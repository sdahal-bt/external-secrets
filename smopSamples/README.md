#### Cluster Setup
1. Follow [ESO Pre-requisites](https://external-secrets.io/latest/introduction/prerequisites/) and install all requried packages

2. Create a Kind Cluster with Local Registry using `ctlptl`
`ctlptl create registry kind-registry --port=5001`
`ctlptl create cluster kind --registry=kind-registry`


#### Installation
1. Build docker image `make docker.build` and load it onto the kind cluster 
`kind load docker-image oci.external-secrets.io/external-secrets/external-secrets:v0.20.4`

2. Package Helm charts `make helm.build` and install them
`helm install external-secrets ./bin/chart/external-secrets.tgz -n external-secrets --create-namespace`

3. Ensure pods are running
`kubectl get pods -n external-secrets`

4. Check if SMoP Provider has been registered in the CRD
`kubectl get crd secretstores.external-secrets.io -o jsonpath='{.spec.versions[?(@.name=="v1")].schema.openAPIV3Schema.properties.spec.properties.provider.properties.smop}' | jq`

5. If the above command doesn't return anything, replace the CRD and try it again
 `kubectl replace -f config/crds/bases/external-secrets.io_secretstores.yaml`

6. Create smop-api-token
`kubectl create secret generic smop-api-token --from-literal=token=<YOUR_PAT_TOKEN>`

7. Create a SMoP SecretStore
`kubectl apply -f smopSamples/smop_secretstore.yaml`

8. Create SMoP External Secret
`kubectl apply -f smopSamples/smop_externalsecret.yaml`



#### Continuous development
1. Build docker image `make docker.build`
2. load image to kind `kind load docker-image oci.external-secrets.io/external-secrets/external-secrets:v0.20.4`
3. Restart deployment `kubectl rollout restart deployment -n external-secrets`
4. Apply external secret `kubectl apply -f smopSamples/smop_externalsecret.yaml`