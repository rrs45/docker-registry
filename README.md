# docker-registry

docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn <<user>> <<pwd>> > auth/htpasswd
  
docker stop registry

docker run -d \
  --restart=always \
  --name registry \
  -v /mnt/configs/registry:/var/lib/registry \
  -v /mnt/configs/certs:/certs \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/var/lib/registry/htpasswd" \
  -e "REGISTRY_HTTP_ADDR=0.0.0.0:5001" \
  -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/client.cert" \
  -e "REGISTRY_HTTP_TLS_KEY=/certs/client.key" \
  -p 5001:5001 \
  registry:latest
  
#Docker registry push
docker login -u <<user>> <<HOSTNAME>>:5001
docker tag vrops:v1 <<HOSTNAME>>:5001/vrops
docker push <<HOSTNAME>>:5001/vrops

#List images from private registry
curl --user 'xxx:xxx' -X GET https://<<HOSTNAME>>:5001/v2/_catalog
curl --user 'xxx:xxx' -X GET https://<<HOSTNAME>>:5001/v2/<<IMAGE_NAME>>/tags/list
curl --user 'xxx:xxx' -X GET https://<<HOSTNAME>>:5001/v2/<<IMAGE_NAME>>/manifests/latest

#Delete image
curl --user 'xxx:xxx' -X DELETE https://<<HOSTNAME>>:5001/v1/repositories/<<IMAGE_NAME>>/tags/latest
  
#Kubernetes integration
kubectl create secret docker-registry regsecret --docker-server=<<HOSTNAME>>:5001 --docker-username=xxxxx --docker-password=xxxxxx --docker-email=xxxxxx
  
 docker service create \
 --network net1 \
 --name registry-service \
 --publish 5001:5001 \
 --constraint node.role == manager \
 --secret site.key \
 --secret site.crt \
	--env  REGISTRY_HTTP_TLS_CERTIFICATE: /run/secrets/site.cert     \
    --env  REGISTRY_HTTP_TLS_KEY: /run/secrets/site.key              \
    --env  REGISTRY_AUTH: htpasswd                                   \
    --env  REGISTRY_AUTH_HTPASSWD_PATH: /mnt/registry/htpasswd       \
    --env  REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm              \
    --env  REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /mnt/registry  \
 regitry:latest
