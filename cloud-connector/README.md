<img src="https://avatars3.githubusercontent.com/u/43526139?s=200&v=4" height="100" width="100" /><img src="https://avatars3.githubusercontent.com/u/15859888?s=200&v=4" width="100" height="100"/>

# KhulnaSoft Security Cloud-Connector Helm Chart

These are Helm charts for installation and maintenance of Khulnasoft Container Security Cloud-Connector

## Contents

- [KhulnaSoft Security Cloud-Connector Helm Chart](#khulnasoft-security-cloud-connector-helm-chart)
  - [Contents](#contents)
  - [Prerequisites](#prerequisites)
    - [Container Registry Credentials](#container-registry-credentials)
  - [Installing the Chart](#installing-the-chart)
    - [Installing Khulnasoft Cloud-Connector from Helm Private Repository](#installing-khulnasoft-cloud-connector-from-helm-private-repository)
  - [Configurable Variables](#configurable-variables)
    - [Cloud-Connector](#cloud-connector)
  - [Issues and feedback](#issues-and-feedback)

## Prerequisites

### Container Registry Credentials

[Link](../docs/imagepullsecret.md)

## Installing the Chart
Follow the steps in this section for production grade deployments. You can either clone khulnasoft-helm git repo or you can add our helm private repository ([https://helm.khulnasoft.com](https://helm.khulnasoft.com))
### Installing Khulnasoft Cloud-Connector from Helm Private Repository

* Add Khulnasoft Helm Repository
```shell
helm repo add khulnasoft-helm https://helm.khulnasoft.com
helm repo update
```

* Check for available chart versions either from [Changelog](./CHANGELOG.md) or by running the below command
```shell
helm search repo khulnasoft-helm/cloud-connector --versions
```

* Install Khulnasoft Cloud-Connector

```shell
helm upgrade --install --namespace khulnasoft khulnasoft-cloud-connector khulnasoft-helm/cloud-connector --set imageCredentials.username=<>,imageCredentials.password=<> --version <>
```

## Configurable Variables

### Cloud-Connector

| Parameter                              | Description                                                                      | Default                 | Mandatory                                    |
|----------------------------------------|----------------------------------------------------------------------------------|-------------------------|----------------------------------------------|
| `imageCredentials.create`              | Set if to create new pull image secret                                           | `false`                 | `YES - New cluster`                          |
| `imageCredentials.name`                | Your Docker pull image secret name                                               | `khulnasoft-registry-secret`  | `YES - New cluster`                          |
| `imageCredentials.repositoryUriPrefix` | Repository uri prefix for dockerhub set `docker.io`                              | `registry.khulnasoft.com`  | `YES - New cluster`                          |
| `imageCredentials.registry`            | Set the registry url for dockerhub set `index.docker.io/v1/`                     | `registry.khulnasoft.com`  | `YES - New cluster`                          |
| `imageCredentials.username`            | Your Docker registry (DockerHub, etc.) username                                  | `khulnasoft-registry-secret`  | `YES - New cluster`                          |
| `imageCredentials.password`            | Your Docker registry (DockerHub, etc.) password                                  | `""`                    | `YES - New cluster`                          |
| `serviceaccount.create`                | Enable to create khulnasoft-sa serviceAccount if it is missing in the environment      | `false`                 | `YES - New cluster`                          |
| `image.repository`                     | The docker image name to use                                                     | `cc-premium`            | `YES`                                        |
| `image.tag`                            | The image tag to use.                                                            | `2022.4`                | `YES`                                        |
| `image.pullPolicy`                     | The kubernetes image pull policy                                                 | `Always`                | `NO`                                         |
| `replicaCount`                         | Kubernetes replica count                                                         | `1`                     | `YES`                                        |
| `authType.tokenAuth`                   | Boolean value to select authentication type as token                             | `true`                  | `YES`                                        |
| `authType.userCreds`                   | Boolean value to select authentication type as user/password                     | `false`                 | `YES`                                        |
| `token`                                | Token value generated from the UI                                                | `""`                    | `YES - authtype selected as token`           |
| `tokenFromSecret.enable`               | Enable to true to load token from existing secret                                | `false`                 | `NO`                                         |
| `tokenFromSecret.secretName`           | Loaded secret name for token                                                     | `""`                    | `NO`                                         |
| `tokenFromSecret.tokenKey`             | Loaded secret token key value                                                    | `""`                    | `NO`                                         |
| `userCreds.username`                   | Admin Username                                                                   | `""`                    | `YES`                                        |
| `userCreds.password`                   | Admin Password                                                                   | `""`                    | `YES`                                        |
| `userCredsFromSecret.enable`           | Enable to true to load user credentials from existing secret                     | `false`                 | `NO`                                         |
| `userCredsFromSecret.secretName`       | Loaded secret name for user credentials                                          | `""`                    | `NO`                                         |
| `userCredsFromSecret.userKey`          | Loaded secret username key value                                                 | `""`                    | `NO`                                         |
| `userCredsFromSecret.passwordKey`      | Loaded secret password key value                                                 | `""`                    | `NO`                                         |
| `healthPort.port`                      | Khulnasoft Cloud Connector Health Port                                                 | `8080`                  | `YES`                                        |
| `tunnels.azure.registryHost`           | Azure container registry host, if ACR is in use for container images             | `""`                    | `NO`                                         |
| `tunnels.azure.registryPort`           | Azure container registry port, if ACR is in use for container images             | `""`                    | `NO`                                         |
| `tunnels.aws.registryHost`             | AWS container registry host, if ECR is in use for container images               | `""`                    | `NO`                                         |
| `tunnels.aws.registryPort`             | AWS container registry type, if ECR is in use for container images               | `ecr`                   | `NO`                                         |
| `tunnels.aws.service.type`             | AWS container registry region, if ECR is in use for container images             | `""`                    | `YES - if AWS ECR in use`                    |
| `tunnels.aws.service.region`           | AWS container registry port, if ECR is in use for container images               | `""`                    | `YES - if AWS ECR in use`                    |
| `tunnels.gcp.registryHost`             | GCP container registry host, if GCR is in use for container images               | `""`                    | `NO`                                         |
| `tunnels.gcp.registryPort`             | Azure container registry port, if GCR is in use for container images             | `""`                    | `NO`                                         |
| `tunnels.jfrog.registryHost`           | JFrog container registry host, if JFrog registry is in use for container images  | `""`                    |                                              |
| `tunnels.jfrog.registryPort`           | JFrog container registry port, if JFrog registry is in use for container images  | `""`                    | `NO`                                         |
| `tunnels.onprem.registryHost`          | OnPrem container registry host, if onPrem registry is in use for container images | `""`                    | `NO`                                         |
| `tunnels.onprem.registryPort`          | OnPrem container registry port, if onPrem registry is in use for container images | `""`                    | `NO`                                         |
| `gateway.host`                         | Gateway host                                                                     | `khulnasoft-gateway-svc.khulnasoft` | `YES`                                        |
| `gateway.port`                         | Gateway port                                                                     | `8443`                  | `YES`                                        |
| `TLS.khulnasoft_verify_enforcer`             | Change it to "1" or "0" for enabling/disabling mTLS between enforcer and envoy   | `0`                     | `YES` <br /> `if TLS.enabled is set to true` |
| `container_securityContext.privileged` | Container security context                                                       | `false`                 | `NO`                                         |
| `resources`                            | Resource requests and limits                                                     | `{}`                    | `NO`                                         |
| `nodeSelector`                         | Kubernetes node selector	                                                        | `{}`                    | `NO`                                         |
| `tolerations`                          | Kubernetes node tolerations	                                                     | `[]`                    | `NO`                                         |
| `podAnnotations`                       | Kubernetes pod annotations                                                       | `{}`                    | `NO`                                         |
| `pdbApiVersion`                        | Override the API Version of PodDisruptionBudget                                  | ``                      | `NO`                                         |
| `extraEnvironmentVars`                 | Is a list of extra environment variables to set in the cc deployments.           | `{}`                    | `NO`                                         |
| `affinity`                             | Kubernetes node affinity                                                         | `{}`                    | `NO`                                         |
| `platform`                             | Platform value, in case of 'openshift', will apply SCC                           | `"`                     | `NO`                                         |
| `extraSecretEnvironmentVars`                             | Allows to add additional environment variables from existing secrets             | `[]`                    | `NO`                                         |


> Note: that `imageCredentials.create` is false and if you need to create image pull secret please update to true, set the username and password for the registry and `serviceAccount.create` is false and if you're environment is new or not having khulnasoft-sa serviceAccount please update it to true.

## Issues and feedback

If you encounter any problems or would like to give us feedback on deployments, we encourage you to raise issues here on GitHub.
