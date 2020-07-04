# aks-deployment
Test Deployment of Azure Kubernetes Service


# How-to

1. Creation of Server principal and Client Principal in AWS IAM or Azure Azure Active Directory(AAD). Follow this article to know how to create one for your org: https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac

2. Spinning up an RBAC enabled Kubernetes Cluster using Azure CLI Command.

3. Prepare Yamls for each tenants(Role and RoleBinding).

3. Define add-on Yamls(resource quota, limit range).

4. DevOps integaraton for deploying applications into provisioned Kubernetes(AKS) cluster. Follow this article for integrating with AzureDevOps(AzDo): https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/aks-template?view=azure-devops