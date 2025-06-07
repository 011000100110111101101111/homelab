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
  HOMEPAGE_VAR_RADARR: 879c5c5403444898a8c6e496dc443802
  HOMEPAGE_VAR_SONARR: f2d8e2c6884c411182376910cf08dab6
  HOMEPAGE_VAR_PROWLARR: 9c1e3a0b93af4d6e81f6b9f4694d6ce5
  HOMEPAGE_VAR_LIDARR: your_lidarr_api_key
  HOMEPAGE_VAR_QBIT: your_qbittorrent_password
  HOMEPAGE_VAR_JELLYFIN: 7ea6b84f776d40d692f6f090031f0cd9
  HOMEPAGE_VAR_JELLYSEER: MTc0NjM5MDk0MzA3OWU1YjFiZjI3LWJhY2YtNGJjOS1hNjZhLWNlODdiZmYzNGQ1OA==
  HOMEPAGE_VAR_PORTAINER-KEY: your_portainer_key
  HOMEPAGE_ALLOWED_HOSTS: "*"
```

Now deploy

```bash
kubeseal -f homepagesecretkey.yaml -w sealedhomepagesecretkey.yaml

# If not already deployed
kubectl create namespace homepage

kubectl create -f sealedhomepagesecretkey.yaml

rm homepagesecretkey.yaml sealedhomepagesecretkey.yaml
```
