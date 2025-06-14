# My Homelab

## Initial Requirements

Install ansible (This is for ubuntu specifically, can also use python pip)

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

## Cluster Bootstrapping

First, configure your inventory. Then run the following to test connectivity.

```bash
ansible -m ping all -i clusterBootstrap/dev-inventory.ini
```

Finally run to install the base cluster, including:

- Connecting the nodes
- Calico as underlying CNI
- Sealed Secrets for secure secrets in repo

```bash
ansible-playbook -i clusterBootstrap/dev-inventory.ini clusterBootstrap/base_install.yaml
```

The cluster is now installed, lets grab the config to manage remotely.

```bash
# On local machine
mkdir ~/.kube
scp username@IP:/home/username/.kube/config ~/.kube/config
```

Okay, now lets inject our needed secrets for the post install. First we need to supply our api key for our domain provider (in my case cloudflare). Yes, alot of this could be automated, feel free to.

```bash
echo -n "apikeyhere" | kubectl create secret generic cloudflare-api-token-secret --dry-run=client --from-file=api-token=/dev/stdin --namespace=cert-manager -o yaml > certapi.yaml

# If you need this, available on macos with brew install kubeseal
kubeseal -f certapi.yaml -w sealedapisecret.yaml

kubectl create namespace cert-manager

kubectl create -f sealedapisecret.yaml

rm certapi.yaml sealedapisecret.yaml
```

Now lets deploy the post stuff,

```bash
ansible-playbook -i clusterBootstrap/dev-inventory.ini clusterBootstrap/post_install.yaml
```

- Longhorn as underlying CSI
- Metal-lb as a baremetal loadbalancer

Now we need to point our DNS for our domain at the haproxy-controller

```bash
kubectl get services -n haproxy-controller
```

Look for the external loadbalancer IP that metallb assigned to it.

Now we need to point our DNS at the haproxy ingress IP, you want to create an A record with your domain name (<www.example.com>) and have it point to the external IP.

The IP to point to is the external IP for the haproxy ingress controller.
Then you can create a CNAME wild card (name would be *) and have it point to your domain name which will point any subdomains to <www.example.com>.

Finally, lets apply our ingresses to access longhorn and argocd. Make sure to update config.yaml for domain name stuff.

```bash
ansible-playbook -i clusterBootstrap/dev-inventory.ini clusterBootstrap/post_install_ingress.yaml
```

Next is the argocd setup

## Initial Pipeline Setup

Once argocd is deployed, you can grab the intial password with the argocd cml tool. Default user is admin.

```bash
argocd admin initial-password -n argocd
```

Now, lets build out the app-of-apps setup. Create a new app on the interface with the following values.

```bash
project: default
source:
  repoURL: https://github.com/011000100110111101101111/homelab
  path: app-in-apps/apps
  targetRevision: HEAD
  directory:
    recurse: true
    jsonnet: {}
destination:
  server: https://kubernetes.default.svc
  namespace: argcocd-sync
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

Essentially, this app is responsible for grabbing any NEW apps and auto deploying them, instead of adding an app to the repo, then having to go manually sync it before it will deploy. For a simple example, once the root app is deployed it is watching anything that gets added to the `app-in-apps` folder. Technically, we could just put raw yaml in there and it would deploy it all, but it would look awful, and be a mess to manage. Instead, we can seperate out our definitions, whether they are yaml, helm, and even seperate repos. Then, we add an `Application` file to the apps folder. This application is pointing to whatever we want to deploy, for example an ngxin server in yaml/examplenginx/ and will auto sync anything in this if we commit to the repo. The only point of the root-app was to be able to deploy our new app (which handles further syncing) from a commit.

## NVIDIA Operator

Needed to manually download the drivers (make sure to update modinit thing)

### Enable time slicing

If you want 1 gpu to server multiple pods, you need the following.

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-slicing-config-all
data:
  any: |-
    version: v1
    flags:
      migStrategy: none
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4
```

Apply

```bash
kubectl create -n gpu-operator -f time-slicing-config-all.yaml
```

Config

```bash
kubectl patch clusterpolicies.nvidia.com/cluster-policy \
    -n gpu-operator --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config-all", "default": "any"}}}}'
```

Confirm

```bash
kubectl get events -n gpu-operator --sort-by='.lastTimestamp'
```

# Creating secrets for other apps

## Can inject secrets with sealed secrets with following method

Wherever it asks for a password in any helm file, can add useSecret, secret and key below it.

```yaml
postgresql:
  password:
    useSecret: true
    secret: my-gitlab-postgresql-password
    key: postgresql-postgres-password
```

Then create the secret (do this on a machine that can connect to cluster over kubectl). Notice where the above vaues are in the below command, as well as setting correct namespace.

```bash
echo -n "password" | kubectl create secret generic my-gitlab-postgresql-password --dry-run=client --from-file=postgresql-postgres-password=/dev/stdin --namespace=gitlab -o yaml > gitlabpassword.yaml

# If you need this, available on macos with brew install kubeseal
kubeseal -f gitlabpassword.yaml -w sealedgitlabpassword.yaml

# If not already deployed
kubectl create namespace gitlab

kubectl create -f sealedgitlabpassword.yaml

rm gitlabpassword.yaml sealedgitlabpassword.yaml
```

# Authentik Deployment

## Temp location for steps

echo -n "password" | kubectl create secret generic my-gitlab-postgresql-password --dry-run=client --from-file=postgresql-postgres-password=/dev/stdin --namespace=gitlab -o yaml > gitlabpassword.yaml
