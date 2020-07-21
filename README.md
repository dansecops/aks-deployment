# aks-deployment
Test Deployment of Azure Kubernetes Service


# Implementation

## 1. Preparation

### Get the latest available Kubernetes version in region:
```VERSION=$(az aks get-versions -l eastus --query 'orchestrators[-1].orchestratorVersion' -o tsv)```

### set variables for AKS
```
AKS_RESOURCE_GROUP="dan-aks-rg-001"
AKS_CLUSTER_NAME="dan-aks-cluster-001"
ACR_NAME="danacr001"
ACR_RESOURCE_GROUP=$AKS_RESOURCE_GROUP
REGION="eastus"
```

## 2. Create an RBAC enabled Cluster


```
# Create Resource Group:
az group create --name $AKS_RESOURCE_GROUP --location eastus

# Create AKS enabled Cluster
az aks create \
    --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME \
    --enable-addons monitoring --kubernetes-version $VERSION \
    --generate-ssh-keys --location $REGION
```

### Deploy Azure Container Registory (ACR)
```az acr create --resource-group dan-aks-rg-001 --name danacr001 --sku Standard --location eastus```

### Grant AKS Service Principal access to ACR
```
# Get the id of the service principal configured for AKS:
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get ACR registry resource ID
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
```

## 3. Create Cosmos DB database

### set variables for Cosmos DB
```
ACCOUNTNAME="dan-cosmos-001" #needs to be lower case \
DATABASENAME="database001" \
CONTAINERNAME="container001" \
COSMOS_RESOURCE_GROUP="dan-cosmosdb-001" \
COSMOSDB_LOCATION="eastus"
```

### Create Resource Group for Cosmos DB
```
az group create -n $COSMOS_RESOURCE_GROUP -l $COSMOSDB_LOCATION
```

### Create a Cosmos account for SQL API
```
az cosmosdb create \
    -n $ACCOUNTNAME \
    -g $COSMOS_RESOURCE_GROUP \
    --default-consistency-level Eventual \
    --locations regionName='East US' failoverPriority=0 isZoneRedundant=False
```

### Create a SQL API database
```
az cosmosdb sql database create \
    -a $ACCOUNTNAME \
    -g $COSMOS_RESOURCE_GROUP \
    -n $DATABASENAME
```
### Create a SQL API container
```
az cosmosdb sql container create \
    -a $ACCOUNTNAME \
    -g $COSMOS_RESOURCE_GROUP \
    -d $DATABASENAME \
    -n $CONTAINERNAME \
    -p '/zipcode' \
    --throughput 400
```

## 4. Receive AKS Credentials
```
az aks get-credentials --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME
```

# Further notes
// Login: 
az aks get-credentials --resource-group contoso-AKS-rg-001 --name aksnonrbac --subscription "Pay-As-You-Go Dev/Test" 

Reset the kubeconfig context using the az aks get-credentials command. In a previous section, you set the context using the cluster admin credentials. The admin user bypasses Azure AD sign in prompts. Without the --admin parameter, the user context is applied that requires all requests to be authenticated using Azure AD

az aks get-credentials --resource-group contoso-AKS-rg-001 --name aksnonrbac --subscription "Pay-As-You-Go Dev/Test"  --overwrite-existing


--overwrite-existing
--admin

// Kubectl
kubectl version
Kubectl cluster-info
kubectl get clusterissuers --all-namespaces

// See the secret content
kubectl -n ns1 get secret widgetsecret -o yaml

// Scaling instances of your service
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10


// Solve issue with OMS agent reporting health logs to Azure Monitor
kubectl get pods --namespace kube-system
kubectl logs --namespace kube-system omsagent-w54dx
kubectl delete pod omsagent-w54dx -n kube-system


// Certificatse

Tls.key and tls.cer or tls.crt
kubectl create secret tls tls-secret --key your_ssl_cert.key --cert your_ssl_cert.crt -n namespaces_name

// Dashboard:
az aks browse --resource-group contoso-rg-01 --name aksnonrbac

Unable to open dashboard you get error saying port conflict

--listen-port 8003
az aks browse --resource-group cmcaks-paas-poc-rgp-001 --name cmcakstst --listen-port 8003

// Kubernetes dashboard is not opening then run below command :
kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard


// Create RBAC AKS cluster:
az aks create --resource-group contoso-AKS-rg-001 --name aksnonrbac --subscription "Pay-As-You-Go Dev/Test" --generate-ssh-keys --aad-server-app-id 254a95*******************86ee --aad-server-app-secret L**************15 --aad-client-app-id 90*********************1677c01 --aad-tenant-id db**********************a20 --location westeurope  --service-principal 254a*********86ee --client-secret ?LEI********************u=_Ft8P15 --node-count 1

// Create RBAC AKS cluster with Availability Zone enabled:
az aks create --resource-group contoso-rg-02 --name aksavz --generate-ssh-keys --aad-server-app-id 254a************86ee --aad-server-app-secret ?LEI3M:Q]************15 --aad-client-app-id 905d1f************31677c01 --aad-tenant-id db8e2*********************b5a20 --location westeurope --service-principal 254a*******************2a86ee --client-secret ?LEI*********u=_Ft8P15 --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --node-vm-size Standard_B2ms --node-count 3 --zones 1 2 3

// Cluster with RBAC AKS cluster with PODs limitation and auotscaler

az aks create --resource-group contoso-rg-02 --name aksavz --generate-ssh-keys --aad-server-app-id 254a************86ee --aad-server-app-secret ?LEI3M:Q]************15 --aad-client-app-id 905d1f************31677c01 --aad-tenant-id db8e2*********************b5a20 --location westeurope --service-principal 254a*******************2a86ee --client-secret ?LEI*********u=_Ft8P15 --node-count 1 --enable-cluster-autoscaler --min-count 1 --max-count 10

// How to check whether current service principal could access ACR
Use following command to check whether the target ACR is listed

az login --service-principal -u <aadClientId> -p <aadClientSecret> -t <tenantId>
az acr list

// Using periscope to get logs from cluster:
az extension add --name aks-preview

az aks kollect -g contoso-rg-01 -n myAKSCluster --storage-account "cloudshellstgcontoso"

az aks list -o table

az configure --defaults group=contoso-rg-01

// Enabling or disabking a Log analytics workspace:  
az resource list

ENABLE
az monitor log-analytics workspace list --resource-group "******" --subscription "*****"

"id": "/subscriptions/**********7/resourceGroups/contoso-rg-02/providers/Microsoft.OperationalInsights/workspaces/test-aks-log-analytics-workspace"

az aks enable-addons -a monitoring -n myAKSCluster1 -g contoso-rg-01 --workspace-resource-id "/subscriptions/3**********7/resourceGroups/contoso-rg-02/providers/Microsoft.OperationalInsights/workspaces/test-aks-log-analytics-workspace"

DISABLE
az aks disable-addons -a monitoring -n "****" -g "*******"

//Get logs
kubectl get events -n monitoring


// To Reset Default RG from Cloudshell
az configure --defaults group= <RG_Name> 
az aks list -o table


// Fetch Service Principal used by your AKS cluster:
az aks list --resource-group contoso-AKS-rg-001
az ad sp list --show-mine -o tsv
az ad sp show --id

// see the charts and version helm repo
helm search repo

// Version of images running in the AKS:
helm list -a --all-namespaces
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
grafana         monitoring      1               2020-03-06 01:13:06.494172712 +0000 UTC deployed        grafana-5.0.5           6.6.2
prometheus1y    prom1yns        1               2020-03-24 12:34:22.348498359 +0000 UTC deployed        prometheus-11.0.3       2.16.0


// Nginx check/Exposing a simple nginx service inside container:
az aks get-credentials --help
To verify the security context of the connection to your cluster, use the kubectl config view command. This should show a non-admin user context.

Using the Azure portal, in Cloud Shell, deploy an NGINX image by using the following command:
kubectl run nginx-11527048 --image=nginx --replicas=1 --port=80
Verify that a Kubernetes pod has been created.

Make the pod available from the internet by using the following command:
kubectl expose deployment nginx-11527048 --port=80 --type=LoadBalancer

To identify the public IP address, use the following command:
kubectl get services

Browse to the public IP address you obtained in the previous step. Verify that the browser displays Welcome to nginx!

// See the secrets content in TLS secrets
kubectl get secrets <secretname> -n <namespace> -o yaml