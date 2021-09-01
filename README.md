
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
    - perform helm uninstall
        ```sh
        helm uninstall consul \
          --namespace consul
        ```
    - delete ns
        ```sh
        kubectl delete ns consul
        ```
    - delete related resources
        ```sh
        kubectl get crds | grep -i 'consul.hashicorp'
        kubectl get clusterroles | grep -i 'consul'
        kubectl get clusterrolebindings | grep -i 'consul'
        ```
        - remove finalizers (if needed)
            ```sh
            kubectl patch crd/$CRD_NAME -p '{"metadata":{"finalizers":[]}}' --type=merge
            ```
    - create new empty consul ns
        ```sh
        kubectl create ns consul
        ```

## Pilchuck
- helm upgrade consul to deploy an IngressGateway in dev2-datahub:
    ```sh
    values_file='pks/consul/charts/dev2-datahub.values.yaml'
    deploy_cluster='dev2-datahub'
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