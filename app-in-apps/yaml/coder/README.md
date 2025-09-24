# Secret deployment prior to app deployment

Create a file (paperlessngx-secrets.yaml) with the following values

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: paperlessngx-secrets
  namespace: paperless
type: Opaque
stringData:
  POSTGRES_PASSWORD: your_postgres_password_here
  PAPERLESS_ADMIN_USER: your_admin_username_here
  PAPERLESS_ADMIN_PASSWORD: your_admin_password_here
```

Now deploy

```bash
# If you have an existing sealed secret
kubectl delete sealedsecrets/paperlessngx-secrets -n paperless

# Seal the secret
kubeseal -f paperlesssecret.yaml -w sealedpaperlesssecret.yaml
# If not already deployed
kubectl create namespace paperless

kubectl create -f sealedpaperlesssecret.yaml

rm paperlesssecret.yaml sealedpaperlesssecret.yaml

kubectl delete sealedsecret -n paperless paperlessngx-secrets
```

New setup

## Deploy vault secrets

```bash
vault kv put kv-v2/paperlessngx POSTGRES_PASSWORD="supersecret" PAPERLESS_ADMIN_USER="admin" PAPERLESS_ADMIN_PASSWORD="adminpass"
```

## Create Vault policy

```bash
vault policy write paperlessngx - <<EOF
path "kv-v2/data/paperlessngx*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

```bash
vault write auth/kubernetes/role/paperlessngx \
    bound_service_account_names=default \
    bound_service_account_namespaces=paperlessngx,database \
    policies=paperlessngx \
    ttl=24h \
    audience=vault
```

## Create vault-crd file

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultAuth
metadata:
  name: paperless-auth
  namespace: paperlessngx
spec:
  method: kubernetes
  mount: kubernetes
  kubernetes:
    role: paperlessngx
    serviceAccount: default
    audiences:
      - vault
```

## Create vault-static-sync.yaml file

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: paperless-secrets
  namespace: paperlessngx
spec:
  vaultAuthRef: paperless-auth
  mount: kv-v2
  type: kv-v2
  path: paperlessngx
  refreshAfter: 10s
  destination:
    create: true
    name: paperlessngx-secrets
  rolloutRestartTargets:
  - kind: Deployment
    name: paperlessngx

---

apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: paperless-db-secrets
  namespace: paperlessngx
spec:
  vaultAuthRef: paperless-auth
  mount: kv-v2
  type: kv-v2
  path: paperlessngx/db
  refreshAfter: 10s
  destination:
    create: true
    name: paperless-db-credentials
  rolloutRestartTargets:
  - kind: Deployment
    name: paperlessngx
```
