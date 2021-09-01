
---

[[_TOC_]]

# Helm upgrade consul to deploy ingress-gateway
- helm upgrade consul to deploy an IngressGateway in dev1-datahub:
    ```sh
    deploy_cluster='dev1-datahub'
    values_file="pks/consul/charts/$deploy_cluster.values.yaml"
    helm upgrade --install consul hashicorp/consul \
      --namespace consul \
      --values "$values_file" \
      --version=0.29.0 \
      --set="global.datacenter=$deploy_cluster" \
      --timeout 10m0s \
      --wait
    ```

# Demo
[official demo](https://www.consul.io/docs/k8s/connect/ingress-gateways)

## Deploying your application to Kubernetes
Now you will deploy a sample application which echoes “hello world”
```sh
kubectl create ns static-gateway
kubectl apply -f pks/consul/test/test-static-server.yaml
```

## Configuring the gateway
```sh
kubectl apply -f pks/consul/manifests/dev1-datahub.ingress-gateway.yaml
kubectl apply -f pks/consul/test/dev.si.yaml
kubectl apply -f pks/consul/test/dev1-datahub.sd.yaml
kubectl -n consul get ingressgateway/dev1-datahub
kubectl -n static-gateway get servicedefaults/static-server
kubectl -n static-gateway get serviceintentions/static-server
```

## Verify in the UI
*NOTE:* this url is for the dev1-datahub cluster  
https://portal.consul.tm.dev1.premera.cloud/ui/dev1-datahub/services


## Connecting to your application
```sh
gateway_ext_ip="$(kubectl -n consul get svc | awk '/consul-dev1-datahub/ {print $4}')"
echo "$gateway_ext_ip"
curl -vkH "Host: static-server.ingress.default.dev1-datahub.consul" "https://$gateway_ext_ip" # tls enabled
```

# Cleanup
```sh
deploy_cluster='dev1-datahub'
kubectl delete -f pks/consul/manifests/"$deploy_cluster".ingress-gateway.yaml
kubectl patch -f pks/consul/manifests/"$deploy_cluster".ingress-gateway.yaml \
  --type json \
  --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
kubectl delete -f pks/consul/test/dev.si.yaml
kubectl patch -f pks/consul/test/dev.si.yaml \
  --type json \
  --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
kubectl delete -f pks/consul/test/$deploy_cluster.sd.yaml
kubectl patch -f pks/consul/test/$deploy_cluster.sd.yaml \
  --type json \
  --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
kubectl delete -f pks/consul/test/test-static-server.yaml
kubectl delete ns static-gateway
```