# My Homelab

## Initial Setup

Once argocd is deployed, you should create a new app on the interface with the following values.

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
