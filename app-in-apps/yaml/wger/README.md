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


## Optionaly post-deploy steps


### Get podname

```bash
kubectl get pods -n wger
```

### Sync over images/videos for workouts/food

```bash
kubectl exec -it my-wger-app-6dd87cdcc8-vtr6p -n wger -- python3 manage.py sync-exercises
kubectl exec -it my-wger-app-6dd87cdcc8-vtr6p -n wger -- python3 manage.py download-exercise-images
kubectl exec -it my-wger-app-6dd87cdcc8-vtr6p -n wger -- python3 manage.py download-exercise-videos
```

### Sync over FULL ingredients list ~1gb
```bash
kubectl exec -it my-wger-app-6dd87cdcc8-vtr6p -n wger -- python3 manage.py sync-ingredients-async &
```