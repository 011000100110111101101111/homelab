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
