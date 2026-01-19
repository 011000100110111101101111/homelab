# Setup

## Deploy vault secrets

```bash
kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh
```

```bash
vault kv put kv-v2/openproject/admin \
  password="admin-password"
```

## Create openproject policy

```bash
vault policy write openproject - <<EOF
path "kv-v2/data/openproject*" {
  capabilities = ["read"]
}
EOF
```

## Create k8s auth role

Add policy to the main default role in the UI

## After adding above role edits, need to restart the main vault pod to update the token, then run following command

```bash
# FIRST UNSEAL THE VAULT
kubectl exec -n vault -it my-vault-0 -- vault operator unseal token1

kubectl exec -n vault -it my-vault-0 -- vault operator unseal token2

kubectl exec -n vault -it my-vault-0 -- vault operator unseal token3

kubectl exec -it pod/my-vault-0 -n vault -- /bin/sh

vault login <token>

vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  token_reviewer_jwt=@/var/run/secrets/kubernetes.io/serviceaccount/token
```
