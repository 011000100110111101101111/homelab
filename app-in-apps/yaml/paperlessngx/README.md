# Secret deployment prior to app deployment

New setup

## Deploy vault secrets

```bash
vault kv put kv-v2/paperlessngx POSTGRES_PASSWORD="supersecret" PAPERLESS_ADMIN_USER="admin" PAPERLESS_ADMIN_PASSWORD="adminpass"
```

## Create paperlessngx policy

```bash
vault policy write paperlessngx - <<EOF
path "kv-v2/data/paperlessngx*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI