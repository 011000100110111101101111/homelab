# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/wger/db username="wger" password="password" database_name="wger" database_host="my-postgresql-wger-rw.database.svc.cluster.local"
```

## Create wger policy

```bash
vault policy write wger - <<EOF
path "kv-v2/data/wger*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI
