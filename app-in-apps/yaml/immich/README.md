# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/immich/db username="immich" password="password"
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

```bash
vault write auth/kubernetes/role/immich \
    bound_service_account_names=default \
    bound_service_account_namespaces=immich,database \
    policies=immich \
    ttl=24h \
    audience=vault
```


## Also need to grab immich locally before deploying

```bash

```