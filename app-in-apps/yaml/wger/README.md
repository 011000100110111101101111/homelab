# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/wger/db username="user" password="password" superuser="superuser" superpass="password"
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
