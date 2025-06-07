# Authentik Documentation

Generate a password for the secret_key and postgresql password

```bash
openssl rand 60 | base64 -w 0
```

Create secret for secret_key

```bash
echo -n "secretkeyhere" | kubectl create secret generic my-authentik-secretkey-password --dry-run=client --from-file=authentik-secretkey-password=/dev/stdin --namespace=authentik -o yaml > authentiksecretkey.yaml

# If you need this, available on macos with brew install kubeseal
kubeseal -f authentiksecretkey.yaml -w sealedauthentiksecretkey.yaml

# If not already deployed
kubectl create namespace authentik

kubectl create -f sealedauthentiksecretkey.yaml

rm authentiksecretkey.yaml sealedauthentiksecretkey.yaml
```

Create sealedsecret for postgres password

```bash
echo -n "postgrespasshere" | kubectl create secret generic my-authentik-postgresql-password --dry-run=client --from-file=authentik-postgres-password=/dev/stdin --namespace=authentik -o yaml > authentikposgres.yaml

# If you need this, available on macos with brew install kubeseal
kubeseal -f authentikposgres.yaml -w sealedauthentikposgres.yaml

kubectl create -f sealedauthentikposgres.yaml

rm authentikposgres.yaml sealedauthentikposgres.yaml
```

## Alternate option with file

You can also create 1 secret file
authentiksecrets.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: authentik-env-secrets
  namespace: authentik
type: Opaque
stringData:
  AUTHENTIK_SECRET_KEY: your_very_secure_key
  AUTHENTIK_POSTGRESQL__PASSWORD: your_postgres_password
```

Seal, deploy and delete

```bash
kubeseal -f authentiksecrets.yaml -w sealedauthentiksecrets.yaml

kubectl create -f sealedauthentiksecrets.yaml

rm authentiksecrets.yaml sealedauthentiksecrets.yaml
```
