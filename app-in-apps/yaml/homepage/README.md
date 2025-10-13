# New setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```


```bash
vault kv put kv-v2/homepage POSTGRES_PASSWORD="supersecret" PAPERLESS_ADMIN_USER="admin" PAPERLESS_ADMIN_PASSWORD="adminpass"
```


## Create homepage policy

```bash
vault policy write homepage - <<EOF
path "kv-v2/data/homepage*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI