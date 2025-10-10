# Make sure to unseal the vault

```bash
kubectl exec -n vault -it my-vault-0 -- vault operator init

#Then use provided keys to unseal
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-1>
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-2>
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-3>

#
kubectl exec --stdin=true --tty=true my-vault-0 -n vault -- /bin/sh

# use root token to login
vault login

# auth
cd tmp

#
vault auth enable -path demo-auth-mount kubernetes

#
vault write auth/demo-auth-mount/config kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

#
vault secrets enable -path=kvv2 kv-v2

#
tee webapp.json <<EOF
path "kvv2/data/webapp/config" {
   capabilities = ["read", "list"]
}
EOF

#
vault policy write webapp webapp.json

# create role
vault write auth/demo-auth-mount/role/role1 \
   bound_service_account_names=demo-static-app \
   bound_service_account_namespaces=app \
   policies=webapp \
   audience=vault \
   ttl=24h

# create secret
vault kv put kvv2/webapp/config username="static-user" password="static-password"

# 
exit
```

## IMPORTANT

```bash
kubectl -n vault create serviceaccount vault-auth
kubectl create clusterrolebinding vault-auth-delegator \
    --clusterrole=system:auth-delegator \
    --serviceaccount=vault:vault-auth
```

Get token

```bash
kubectl -n vault create token vault-auth
```

```bash
vault write auth/kubernetes/config \
    kubernetes_host="https://10.96.0.1:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    token_reviewer_jwt=<token-from-step-2>
```

## For using default SA across all namespaces to reduce vaultauths

```bash
vault write auth/kubernetes/role/my-vault-role \
  bound_service_account_names=default \
  bound_service_account_namespaces='*' \
  policies=pastefy \
  ttl=24h \
  audience=vault
```