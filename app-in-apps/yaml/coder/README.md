# Secret deployment prior to app deployment

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -- /bin/sh
```

```bash
vault kv put kv-v2/coder/db_url url="postgres://coder:qdu_bke!fhm0ntm*YAH@postgres.database.svc.cluster.local:5432/coder?sslmode=disable"
vault kv put kv-v2/coder/db username="coder" password="qdu_bke!fhm0ntm*YAH"
```

## Create Vault policy

```bash
vault policy write coder - <<EOF
path "kv-v2/data/coder*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

```bash
vault write auth/kubernetes/role/coder \
    bound_service_account_names=default \
    bound_service_account_namespaces=coder,database \
    policies=coder \
    ttl=24h \
    audience=vault
```
