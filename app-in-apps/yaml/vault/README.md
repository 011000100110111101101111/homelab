# Make sure to unseal the vault

```bash
kubectl exec -n vault -it my-vault-0 -- vault operator init

#Then use provided keys to unseal
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-1>
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-2>
kubectl exec -n vault -it my-vault-0 -- vault operator unseal <unseal-key-3>
```
