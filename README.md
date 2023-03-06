### 一、安装registry.

```javascript
helm repo add twuni https://helm.twun.io
helm repo update
yum install -y httpd-tools
htpasswd -bBc /tmp/htpasswd admin Test@123
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
  
  %2F
curl -u admin:Test@123 http://127.0.0.1:5000/v2/_catalog
```
