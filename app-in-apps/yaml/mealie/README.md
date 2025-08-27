# Secret deployment prior to app deployment

Create a file (postgressecret.yaml) with the following values

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
  namespace: mealie
type: Opaque
stringData:
  postgres-password: postgres_password
```

Now deploy

```bash
# If you have an existing secret
kubectl delete sealedsecrets/postgres-credentials -n mealie

kubeseal -f postgressecret.yaml -w sealedpostgressecret.yaml

# If not already deployed
kubectl create namespace mealie

kubectl create -f sealedpostgressecret.yaml

rm postgressecret.yaml sealedpostgressecret.yaml
```
