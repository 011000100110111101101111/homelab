# Secret deployment prior to app deployment

Create a file (homepagesecretkey.yaml) with the following values

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: homepage-secrets
  namespace: homepage
type: Opaque
stringData:
  HOMEPAGE_VAR_RADARR: radarr_key
  HOMEPAGE_VAR_SONARR: sonarr_key
  HOMEPAGE_VAR_PROWLARR: prowlarr_key
  HOMEPAGE_VAR_LIDARR: your_lidarr_api_key
  HOMEPAGE_VAR_QBIT: your_qbittorrent_password
  HOMEPAGE_VAR_JELLYFIN: jellyfin_key
  HOMEPAGE_VAR_JELLYSEER: jellyseer_key
  HOMEPAGE_VAR_PORTAINER-KEY: your_portainer_key
  HOMEPAGE_ALLOWED_HOSTS: "*"
```

Now deploy

```bash
# If you have an existing secret
kubectl delete sealedsecrets/homepage-secrets -n homepage

kubeseal -f homepagesecretkey.yaml -w sealedhomepagesecretkey.yaml

# If not already deployed
kubectl create namespace homepage

kubectl create -f sealedhomepagesecretkey.yaml

rm homepagesecretkey.yaml sealedhomepagesecretkey.yaml
```
