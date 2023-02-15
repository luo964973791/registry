### 一、安装registry.

```javascript
helm install docker-registry \
  --namespace container-registry --create-namespace \
  --set replicaCount=2 \
  --set persistence.enabled=true \
  --set persistence.size=6Gi \
  --set service.type=LoadBalancer \
  --set persistence.deleteEnabled=true \
  --set persistence.storageClass=local-path \
  --set secrets.htpasswd=$(cat /tmp/htpasswd) \
  twuni/docker-registry
```
