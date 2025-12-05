# Secret deployment prior to app deployment

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/metabase/db username="metabase" password="password" host="my-postgresql-metabase-rw.database.svc.cluster.local"
```

## Create metabase policy

```bash
vault policy write metabase - <<EOF
path "kv-v2/data/metabase*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI