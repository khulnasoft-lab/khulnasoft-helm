<img src="https://avatars3.githubusercontent.com/u/43526139?s=200&v=4" height="100" width="100" /><img src="https://avatars3.githubusercontent.com/u/15859888?s=200&v=4" width="100" height="100"/>

# Overview


This page contains instructions for deploying Khulnasoft Enterprise in a Kubernetes cluster, using the [Helm package manager](https://helm.sh/).

Refer to the Khulnasoft Enterprise product documentation for the broader context: [Kubernetes with Helm Charts](https://docs.khulnasoft.com/v2022.4/docs/kubernetes-with-helm).

## Contents

- [Overview](#overview)
  - [Contents](#contents)
  - [Helm charts](#helm-charts)
- [Deployment instructions](#deployment-instructions)
    - [(Optional) Add the Khulnasoft Helm repository](#optional-add-the-khulnasoft-helm-repository)
        - [For Helm 2.x](#for-helm-2x)
        - [For Helm 3.x](#for-helm-3x)
    - [Deploy the Helm charts](#deploy-the-helm-charts)
      - [Error 2](#error-2)
      - [Error 3](#error-3)
- [Quick-start deployment (not for production purposes)](#quick-start-deployment-not-for-production-purposes)
- [Issues and feedback](#issues-and-feedback)

## Helm charts

This repository includes the following charts; they can be deployed separately:

| Chart                               | Description                                                                                                                                                   | Latest Chart Version |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| [Server](server/)                   | Deploys the Console, Database, and Gateway components; optionally deploys Envoy component                                                                     | 2022.4.26            |
| [Enforcer](enforcer/)               | Deploys the Khulnasoft Enforcer daemonset                                                                                                                           | 2022.4.24            |
| [Scanner](scanner/)                 | Deploys the Khulnasoft Scanner deployment                                                                                                                           | 2022.4.10            |
| [KubeEnforcer](kube-enforcer/)      | Deploys KhulnaSoft KubeEnforcer                                                                                                                                     | 2022.4.55            |
| [Gateway](gateway)                  | Deploys the Khulnasoft Standalone Gateway                                                                                                                           | 2022.4.14            |
| [Tenant-Manager](tenant-manager/)   | Deploys the Khulnasoft Tenant Manager                                                                                                                               | 2022.4.0             |
| [Cyber Center](cyber-center/)       | Deploys Khulnasoft CyberCenter offline for air-gap environment                                                                                                      | 2022.4.6             |
| [Cloud Connector](cloud-connector/) | Deploys the Khulnasoft Cloud Connector                                                                                                                              | 2022.4.5             |
| [QuickStart](khulnasoft-quickstart/)      | Not for production use (see [below](#quick-start-deployment-not-for-production-purposes)). Deploys the Console, Database, Gateway and KubeEnforcer components | 2022.4.1             |
| [Codesec-Agent](codesec-agent/)     | Argon Broker Deployment                                                                                                                                       | 1.2.11               |

# Deployment instructions

Khulnasoft Enterprise deployments include the following components:
- Server (Console, Database, and Gateway)
- Enforcer
- KubeEnforcer
- Scanner (Optional)

Follow the steps in this section for production-grade deployments. You can either clone the khulnasoft-helm git repo or you can add our Helm private repository ([https://helm.khulnasoft.com](https://helm.khulnasoft.com)).

### (Optional) Add the Khulnasoft Helm repository

1. Add the Khulnasoft Helm repository to your local Helm repos by executing the following command:
 ```shell
 helm repo add khulnasoft-helm https://helm.khulnasoft.com
 helm repo update
 ```

2. Search for all components of the latest version in our Khulnasoft Helm repository

##### For Helm 2.x
```shell
helm search khulnasoft-helm
# Examples
helm search khulnasoft-helm --versions
helm search khulnasoft-helm --version 2022.4
```

##### For Helm 3.x
```shell
helm search repo khulnasoft-helm
# Examples
helm search repo khulnasoft-helm --versions
helm search repo khulnasoft-helm --version 2022.4
```

Example output:
```csv
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
khulnasoft-helm/codesec-agent         1.2.11          2022.4          A Helm chart for the Argon Broker Deployment
khulnasoft-helm/cloud-connector       2022.4.4        2022.4          A Helm chart for Khulnasoft Cloud-Connector
khulnasoft-helm/cyber-center          2022.4.6        2022.4          A Helm chart for Khulnasoft CyberCenter
khulnasoft-helm/enforcer              2022.4.24       2022.4          A Helm chart for the Khulnasoft Enforcer
khulnasoft-helm/kube-enforcer         2022.4.55       2022.4          A Helm chart for the KhulnaSoft KubeEnforcer Starboard
khulnasoft-helm/gateway               2022.4.14       2022.4          A Helm chart for the Khulnasoft Gateway
khulnasoft-helm/scanner               2022.4.10       2022.4          A Helm chart for the Khulnasoft Scanner CLI component
khulnasoft-helm/server                2022.4.26       2022.4          A Helm chart for the Khulnasoft Console components
khulnasoft-helm/tenant-manager        2022.4.1        2022.4          A Helm chart for the Khulnasoft Tenant Manager
```

### Deploy the Helm charts

1. Add Khulnasoft Helm Repository
   ```
   helm repo add khulnasoft-helm https://helm.khulnasoft.com
   helm repo update
   ```
   Check for available chart versions either from [Changelog](./CHANGELOG.md) or by running the below command.
    ```
   helm search repo khulnasoft-helm/enforcer --versions
   ```

   Create the `khulnasoft` namespace.
    ```shell
    kubectl create namespace khulnasoft
    ```
   Create `khulnasoft-registry` secret
   ```
   kubectl create secret docker-registry khulnasoft-registry-secret \
    --docker-server=registry.khulnasoft.com \
    --docker-username=$YOUR_REGISTRY_USER \
    --docker-password=$YOUR_REGISTRY_PASSWORD \
    -n khulnasoft
   ```
2. Deploy the [**Server**](server/) chart.
    ```
   helm upgrade --install --namespace khulnasoft khulnasoft khulnasoft-helm/server --version $VERSION \
   --set imageCredentials.create=false \
   --set global.platform=$PLATFORM
    ```
3. Deploy the [**Enforcer**](enforcer/) chart.
    ```
   helm upgrade --install --namespace khulnasoft khulnasoft-enforcer khulnasoft-helm/enforcer --version $VERSION \
   --set imageCredentials.create=false \
   --set global.platform=$PLATFORM
    ```
4. Deploy the [**KubeEnforcer**](kube-enforcer/) chart.
    ```
   helm upgrade --install --namespace khulnasoft kube-enforcer khulnasoft-helm/kube-enforcer --version $VERSION \
    --set global.platform=$PLATFORM \
    --set certsSecret.autoGenerate=true
    ```
5. (Optional) Deploy the [**Scanner**](scanner/) chart.
    ```
   helm upgrade --install --namespace khulnasoft scanner khulnasoft-helm/scanner --version $VERSION \
    --set user=$KHULNASOFT_CONSOLE_USERNAME \
    --set password=$KHULNASOFT_CONSOLE_PASSWORD
    ```
6. Gateway is Deployed by default with Server chart, advanced Gateway Deployment options can be found [**Here**](gateway/).
7. (Optional) Deploy the [**TenantManager**](tenant-manager/) chart.
    ```
   helm upgrade --install --namespace khulnasoft tenant-manager khulnasoft-helm/tenant-manager --version $VERSION \
   --set platform=$PLATFORM
    ```
8. (Optional) Deploy the [**Cyber-Center**](cyber-center/) chart.
    ```
   helm upgrade --install --namespace khulnasoft khulnasoft-cyber-center khulnasoft-helm/cyber-center --version $VERSION \
   --set imageCredentials.create=false
    ```
9. (Optional) Deploy the [**Cloud-Connector**](cloud-connector) chart.
    ```
    helm upgrade --install --namespace khulnasoft khulnasoft-cloud-connector khulnasoft-helm/cloud-connector --version $VERSION \
   --set userCreds.username=$KHULNASOFT_CONSOLE_USERNAME \
   --set userCreds.password=$KHULNASOFT_CONSOLE_PASSWORD \
   --set authType.tokenAuth=false \
   --set authType.userCreds=true
    ```
10. Access the Khulnasoft UI in browser with {{ .Release.Name }}-console-svc service and port, to check the service details:
      ```shell
      kubectl get svc -n khulnasoft
      ```
    * Example:
        * http://< Console IP/DNS >:8080* (default access without SSL) or
        * https://< Console IP/DNS >:443* (If SSL configured to console component in server chart)
### Troubleshooting

**This section not all-inclusive. It describes some common issues that we have encountered during deployments.**

#### Error 1

* Error message: **UPGRADE/INSTALL FAILED, configmaps is forbidden.**
* Example:

```shell
Error: UPGRADE FAILED: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list configmaps in the namespace "kube-system"
```

* Solution: Create a service account for Tiller to utilize.
```shell
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
helm init --service-account tiller --upgrade
```

#### Error 2

* Error message: **No persistent volumes available for this claim and no storage class is set.**
* Solution: Most managed Kubernetes deployments do NOT include all possible storage provider variations at setup time. Refer to the [official Kubernetes guidance on storage classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) for your platform.
  For more information see the [storage documentation](docs/storage.md).

#### Error 3

* Error message: When executing `kubectl get events -n khulnasoft` you might encounter either **No persistent volumes available for this claim and no storage class is set** or
  **PersistentVolumeClaim is not bound**.
* Solution: If you encounter either of these errors, you need to create a persistent volume prior to chart deployment with a generic or existing storage class. Specify `db.persistence.storageClass` in the values.yaml file. A sample file using `khulnasoft-storage` is included in the repo.

```shell
kubectl apply -f pv-example.yaml
```

# Quick-start deployment (not for production purposes)

Quick-start deployments are fast and easy.
They are intended for deploying Khulnasoft Enterprise for non-production purposes, such as proofs-of-concept (POCs) and environments intended for instruction, development, and test.

Use the [**khulnasoft-quickstart**](khulnasoft-quickstart) chart to

1. Clone the GitHub repository
  ```shell
  git clone https://github.com/khulnasoft-lab/khulnasoft-helm.git
  cd khulnasoft-helm/
  ```

2. Create the `khulnasoft` namespace.
  ```shell
  kubectl create namespace khulnasoft
  ```

3. Deploy khulnasoft-quickstart chart
  ```shell
  helm upgrade --install --namespace khulnasoft khulnasoft ./khulnasoft-quickstart --set imageCredentials.username=<>,imageCredentials.password=<>
  ```

# Issues and feedback

If you encounter any problems or would like to give us feedback on deployments, we encourage you to raise issues here on GitHub.
