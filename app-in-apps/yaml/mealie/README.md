# New setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

## Create secret in vault
```bash
vault kv put kv-v2/mealie/db password="vpruptpErbtmYgaDGLvpErisB4PYsnuY"
```

## Create mealie policy

```bash
vault policy write mealie - <<EOF
path "kv-v2/data/mealie*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI