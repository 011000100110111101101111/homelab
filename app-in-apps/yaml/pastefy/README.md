# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/pastefy/db username="pastefy" password="password" database_name="pastefy-dbs" database_host="my-postgresql-pastefy-rw.database.svc.cluster.local"
```

## Create immich policy

```bash
vault policy write pastefy - <<EOF
path "kv-v2/data/pastefy*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

```bash
vault write auth/kubernetes/role/pastefy \
    bound_service_account_names=default \
    bound_service_account_namespaces=pastefy,database \
    policies=pastefy \
    ttl=24h \
    audience=vault
```
