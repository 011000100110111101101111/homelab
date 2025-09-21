# New setup

## Deploy vault secrets

```bash
vault kv put kv-v2/homepage POSTGRES_PASSWORD="supersecret" PAPERLESS_ADMIN_USER="admin" PAPERLESS_ADMIN_PASSWORD="adminpass"
OR
Create it in vault itself
```

## Create Vault policy

```bash
vault policy write homepage - <<EOF
path "kv-v2/data/homepage*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

```bash
vault write auth/kubernetes/role/homepage \
    bound_service_account_names=default \
    bound_service_account_namespaces=homepage \
    policies=homepage \
    ttl=24h \
    audience=vault
```
