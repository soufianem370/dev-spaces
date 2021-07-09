### Azure Dev Spaces is retired as of May 15, 2021.

### commandes azure

```
docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh soufianem370/kube-tools-azure-helm-terraform:latest
```
#### créer cluster AKS
```
az login
az group create --name kops-test --location eastus
{
  "id": "/subscriptions/964df7ca-3ba4-48b6-a695-1ed9db5723f8/resourceGroups/kops-test",
  "location": "eastus",
  "managedBy": null,
  "name": "kops-test",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}

az group list
export RG=
az aks create --resource-group $RG --name cluster01 --node-count 2  --generate-ssh-keys --enable-managed-identity
```
#### arret/demarrer cluster
```
az aks stop --name cluster01 --resource-group kops-test
az aks start --name cluster01 --resource-group kops-test
az aks update --attach-acr acrsoufiane2 --name cluster01 --resource-group kops-test
```
#### créer un acr mode graphique

pour tester votre acr:

sur l'interface ACR ===>clé acces==> activer utilisateur administrateur

#### tester la connexion a votre ACR

```
docker login acrsoufiane2.azurecr.io
docker tag mcr.microsoft.com/azure-cli:2.6.0 acrsoufiane2.azurecr.io/acrsoufiane2/azure-cli:latest
docker push acrsoufiane2.azurecr.io/acrsoufiane2/azure-cli:latest
```

===>importe image from an other registry docker
```
az acr import  -n kopstest --source docker.io/soufianem370/kube-tools-azure-helm-terraform:latest --image kube-tools-az:v1
```
===>builder avec les commandes az

```
az acr build --image webfrontend:v1 --registry kopstest --file Dockerfile .
```

https://github.com/sabbour/aks-gitops/blob/master/_docs/provision-infrastructure.md ===> aprobation entre aks et acr (pas encore tester)

```
az acr create --resource-group $RG --name "aksacrsoufiane" --sku "Standard" --location "eastus"
export AKS_SP_ID=$(az aks show -g $RG -n cluster01 --query "servicePrincipalProfile.clientId" -o tsv)
export ACR_RESOURCE_ID=$(az acr show -g $RG -n acrsoufiane2 --query "id" -o tsv)
az role assignment create --assignee $AKS_SP_ID --scope $ACR_RESOURCE_ID --role contributor
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl
```
# example application nodejs build ==< push vers ACR ==> deploy vers AKS with helm

### lancer image docker d'envirennement azure

```
docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh soufianem370/kube-tools-azure-helm-terraform:latest
```
```
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/nodejs/getting-started/webfrontend
```
Create a Dockerfile
-----------------------
```
FROM node:latest

WORKDIR /webfrontend

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80
CMD ["node","server.js"]
```

==> builder votre image docker via les commandes az acr 

```
az acr build --image webfrontend:v1 --registry kopstest --file Dockerfile .
```
nous avons une image stoquer sur ACR kopstest.azurecr.io/webfrontend:v1

==>editer le fichier values de votre chart

```
image:
  repository: kopstest.azurecr.io/webfrontend
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1"
```
```
# kubectl create ns webfrontend
# helm install webfrontend . -n webfrontend
```
Azure Dev Spaces is retired as of May 15, 2021. Developers should use [Bridge to Kubernetes](https://aka.ms/bridge-to-k8s-ga), a client developer tool.

The purpose of Azure Dev Spaces was about easing developers into developing on Kubernetes. A significant tradeoff in the approach of Azure Dev Spaces was putting extra burden on developers to understand Docker and Kubernetes configurations as well as Kubernetes deployment concepts. Over time, it also became clear that the approach of Azure Dev Spaces did not effectively decrease the speed of inner loop development on Kubernetes. Bridge to Kubernetes effectively decreases the speed of inner loop development and avoids unnecessary burden on developers.

The core mission remains unchanged: Build the best developer experiences to develop, test, and debug microservice code in the context of the larger application.

### How does this affect me?
Azure Dev Spaces is no longer supported, and all projects using Azure Dev Spaces will no longer function.

### What action should I take?
Transition to [Bridge to Kubernetes](https://aka.ms/bridge-to-k8s-ga). More information on migration options is available [here](https://aka.ms/migrate-to-bridge).
If you have migration questions, or need technical assistance, please email [BridgeToKubernetes-Talk@microsoft.com](BridgeToKubernetes-Talk@microsoft.com).

# Contributing
This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
