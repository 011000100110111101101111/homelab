# Secret deployment prior to app deployment

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/coder/db_url url="postgres://coder:password@postgres.database.svc.cluster.local:5432/coder?sslmode=disable"
vault kv put kv-v2/coder/db username="coder" password="password"
```

## Create coder policy

```bash
vault policy write coder - <<EOF
path "kv-v2/data/coder*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI