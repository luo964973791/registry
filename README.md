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
  
curl -u admin:Test@123 http://127.0.0.1:5000/v2/_catalog



#docker部署
mkdir -p  /data/registry/auth
mkdir -p  /data/registry/config
docker run --rm httpd:alpine htpasswd -Bbn admin 123456 >> /data/registry/auth/htpasswd
cat << EOF > /data/registry/config/config.yml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
threshold: 3
EOF  
   
docker run -d -p 5000:5000 --restart=always  --name=registry \
-v /data/registry/config/:/etc/docker/registry/ \
-v /data/registry/auth/:/auth/ \
-e "REGISTRY_AUTH=htpasswd" \
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
-v /data/registry/:/var/lib/registry/ \
registry


#registry-web ui
docker run -it -p 8082:80 --name docker-registry-ui \
  -e REGISTRY_URL=http://172.27.0.8:5000 \
  -e CATALOG_ELEMENTS_LIMIT="1000" \
  joxit/docker-registry-ui:static



#删除镜像步骤.
curl -u "admin:123456" -X GET http://172.27.0.8:5000/v2/nginx/tags/list
echo -n "admin:123456" | base64
#这里的Basic YWRtaW46MTIzNDU2填echo -n "admin:123456" | base64加密的字符串.
curl -i -H "Authorization: Basic YWRtaW46MTIzNDU2" -H "Accept: application/vnd.docker.distribution.manifest.v2+json" http://172.27.0.8:5000/v2/nginx/manifests/20240117-151117 | grep "Docker-Content-Digest:"
#这里的sha256:c13cf1c70064e1691ee7f33763ab6874418a81cc59cef365b158a7470a784791填上面的输出结果.
curl -X "DELETE" -H "Authorization: Basic YWRtaW46MTIzNDU2" -H "Accept: application/vnd.docker.distribution.manifest.v2+json" http://172.27.0.8:5000/v2/nginx/manifests/sha256:c13cf1c70064e1691ee7f33763ab6874418a81cc59cef365b158a7470a784791
```
