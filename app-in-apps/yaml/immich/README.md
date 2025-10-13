# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/immich/db \
  DB_HOSTNAME="immich-database-rw.database.svc.cluster.local" \
  DB_PORT="5432" \
  DB_USERNAME="immich" \
  DB_PASSWORD="password" \
  DB_DATABASE_NAME="immichdb"
```

## Create immich policy

```bash
vault policy write immich - <<EOF
path "kv-v2/data/immich*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI