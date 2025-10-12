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