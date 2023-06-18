# Azure DevOps - template-aks-deployment-pipeline

# Pre-Requisites

## GitHub Service Connection

A GitHub service connection must be configured in the Azure DevOps project being used with this template. Alternatively this can be forked into your Azure DevOps organisation and referenced internally.

## Azure DevOps Environment

This template uses an Azure DevOps environment for the deployment stage, you should ensure that appropriate environments exist in your project with approvals configured as neccessary.

## Azure DevOps Service Connections

Service Connections to Azure Container Registry and Azure Kubernetes Service must exist in the Azure DevOps project using this pipeline template and are used for the deployment.

**NOTE**: Even if you're not using any private images hosted on Azure Container Registry, a service connection must still be provided.

# Validation

The following tooling is included in this pipeline template for validation of deployments:

## Kubeconform - YAML Manifest Validation

During the prepare stage of this pipeline template, [kubeconform](https://github.com/yannh/kubeconform) is used to check that the manifest being provided are valid and will flag any issues to be corrected before proceeding to the deployment stage.

## Polaris - Security Best Practice Checks

During the prepare stage of this pipeline template, [polaris](https://github.com/FairwindsOps/polaris) is used to check that the deployment meets specified security best practises at time of deployment.

Checks are defined in [polaris_checks.yml](./jobs/polaris_checks.yml), categorised into:
- **Security**
- **Reliability**
- **Efficiency**

Checks have defined severities of:
- **Danger** - The pipeline will fail if these checks are not compliant
- **Warning** - A warning will be displayed under the pipeline job which will indicate how your deployment could be improved in the 3 categories
- **Success** - The check is compliant for the deployment.

## Custom Checks

Polaris supports the creation of custom checks for evaluation. The following are included in this pipeline template:

microSegmentation - A 'NetworkPolicy' resource must exist to restrict connectivity between namespaces recommended for logical separation of workloads.

# Usage

## YAML Manifest Deployment

```
resources:
  repositories:
  - repository: template-aks-deployment-pipeline
    type: github
    name: lcondliffe/template-aks-deployment-pipeline
    endpoint: github.com_example_service_connection
    ref: main

trigger:
  paths:
    include:
    - './manifests/*.yml'

pool: example

stages:
- template: main.yml@template-aks-deployment-pipeline
  parameters:
    environment: example
    deployment_type: yaml
    aks_service_connection: example-aks-service-connection
    acr_service_connection: example.azurecr.io
    namespace: replace
    manifests: './manifests/*.yml'
```

## Helm Chart Deployment

```
resources:
  repositories:
  - repository: template-aks-deployment-pipeline
    type: github
    name: lcondliffe/template-aks-deployment-pipeline
    endpoint: github.com_example_service_connection
    ref: main

trigger: none

pool: example

stages:
- template: main.yml@template-aks-deployment-pipeline
  parameters:
    environment: example
    deployment_type: helm
    aks_service_connection: example-service-connection
    acr_service_connection: example.azurecr.io
    namespace: example
    helm_repo_name: example
    helm_repo_url: https://charts.example.io
    helm_chart_name: example/chart
    helm_chart_version: 1.0.0
    helm_release_name: example
    helm_values_file: ./values.yml
```

## Kustomize Deployment

```
resources:
  repositories:
  - repository: template-aks-deployment-pipeline
    type: github
    name: lcondliffe/template-aks-deployment-pipeline
    endpoint: github.com_example_service_connection
    ref: main

trigger: none

pool: example

stages:
- template: main.yml@template-aks-deployment-pipeline
  parameters:
    environment: example
    deployment_type: kustomize
    aks_service_connection: example-service-connection
    acr_service_connection: example.azurecr.io
    namespace: example
    kustomize_extra_yaml_template: example-local-repo.yml
```

## Create Azure Container Registry imagePullSecret

If the Kubernetes deployment relies on private container images in Azure Container Registry, an **imagePullSecret** is required in the target namespace to enable authentication to the registry to pull those images.

If this is required for the deployment include the parameter:

`create_acr_secret: true`

This will create a secret named **acr** that can be used to pull images from the container registry specified in *acr_service_connection*

## Creating Kubernetes Secrets

Some applications require secrets to exist alongside them for particular functions such as authentication to external services, local account passwords etc.

This can be achieved using Azure DevOps secret variables. Create a varible group and a secret variable, include the group in the pipeline definition deployment parameters and specify the secret value as shown in the example below:

```
    create_literal_secret: true
    secret:
    - name: secured
      value: $(securevar)
    - name: plaintext
      value: somethingnotsecure

```

You can also create Kubernetes secrets from secure files in the Azure DevOps project library

```
    create_file_secret: true
    secure_file:
    - secret_name: secret01
      file_name: securefilename
      
```

# References

https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/kubernetes-manifest-v0?view=azure-pipelines

https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/helm-deploy-v0?view=azure-pipelines

https://github.com/yannh/kubeconform

https://github.com/FairwindsOps/polaris

https://polaris.docs.fairwinds.com/

https://polaris.docs.fairwinds.com/customization/custom-checks/#checking-cpu-and-memory

https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/resources-repositories-repository?view=azure-pipelines#examples

