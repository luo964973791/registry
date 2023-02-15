### 一、安装registry.

```javascript
helm repo add twuni https://helm.twun.io
helm repo update
echo 'admin:Test@123' > /tmp/htpasswd
helm install docker-registry \
  --namespace container-registry --create-namespace \
  --set replicaCount=2 \
  --set persistence.enabled=true \
  --set persistence.size=6Gi \
  --set service.type=LoadBalancer \
  --set persistence.deleteEnabled=true \
  --set persistence.storageClass=local-path \
  twuni/docker-registry
```
