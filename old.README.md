
---

[[_TOC_]]

# Helm upgrade consul to deploy ingress-gateway
## ECS
- [values file](https://premeraservices.visualstudio.com/DES/_git/pks-consul-ingress-poc?path=%2Fconsul-Ent-Second-ServiceMeshDC2.yml)
- helm install with:
    ```sh
    helm upgrade --install -f pks/consul/charts/consul-Ent-Second-ServiceMeshDC2.yml --set="global.datacenter=test1-ecs” consul hashicorp/consul --version=0.29.0 -n consul --wait --timeout 10m0s
    ```
    - "the version of consul we are using is tied to chart version 0.29.0"
- revert to helm install with (original ECS release)[https://premeraservices.visualstudio.com/DES/_build?definitionId=677&_a=summary] (runs every 15 mins or can be manually triggered)

### Fully remove consul and all it's resources
- delete and patch previous consul stuff
    ```sh
    kubectl delete -f service-defaults.yaml
    kubectl delete -f service-intentions.yaml
    kubectl delete -f ingress-gateway.yaml
    kubectl patch -f service-defaults.yaml \
      --type json \
      --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
    kubectl patch -f service-intentions.yaml \
      --type json \
      --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
    kubectl patch -f ingress-gateway.yaml \
      --type json \
      --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
    ```
- perform helm uninstall
    ```sh
    kubectl delete podsecuritypolicies.policy/consul-tls-init-cleanup
    helm uninstall consul \
      --namespace consul
    ```
- delete consul pvcs that helm uninstall doesn't
    ```sh
    kubectl delete pvc -l chart=consul-helm -A
    ```
- delete related resources
    ```sh
    crds=()
    kubectl get crds | awk '/consul.hashicorp/ {print $1}' | while IFS="" read -r line; do crds+=("$line"); done
    for crd in "${crds[@]}"; do
      kubectl patch crd/"$crd" -p '{"metadata":{"finalizers":[]}}' --type=merge
      kubectl delete crd "$crd"
    done
    kubectl get clusterroles | awk '/consul/ {print $1}' | xargs -L1 kubectl delete clusterroles
    kubectl get clusterrolebindings | awk '/consul/ {print $1}' | xargs -L1 kubectl delete clusterrolebindings
    ```
- delete ns
    ```sh
    kubectl delete ns consul
    ```
- create new empty consul ns
    ```sh
    kubectl create ns consul
    ```

## Pilchuck
- helm upgrade consul to deploy an IngressGateway in dev2-datahub:
    ```sh
    deploy_cluster='dev2-datahub'
    values_file="pks/consul/charts/$deploy_cluster.values.yaml"
    helm upgrade --install consul hashicorp/consul \
      --namespace consul \
      --values "$values_file" \
      --version=0.29.0 \
      --set="global.datacenter=$deploy_cluster" \
      --timeout 10m0s \
      --wait
    ```

# Demos
- [official demo](https://www.consul.io/docs/k8s/connect/ingress-gateways)
- [pilchuck demo](https://pbc.visualstudio.com/Premera/_git/DataHub_Platform?version=GBmaster&path=%2Fpks%2Fconsul%2Fdemo%2FREADME.demo.md&_a=preview)