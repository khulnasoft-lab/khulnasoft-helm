<img src="https://avatars3.githubusercontent.com/u/43526139?s=200&v=4" height="100" width="100" /><img src="https://avatars3.githubusercontent.com/u/15859888?s=200&v=4" width="100" height="100"/>

# KhulnaSoft Security Scanner Helm Chart

These are Helm charts for installation and maintenance of Khulnasoft Container Security Platform Scanner CLI.

## Contents

- [KhulnaSoft Security Scanner Helm Chart](#khulnasoft-security-scanner-helm-chart)
  - [Contents](#contents)
  - [Prerequisites](#prerequisites)
    - [Container Registry Credentials](#container-registry-credentials)
    - [Scanner User Credentials](#scanner-user-credentials)
  - [Installing the Chart](#installing-the-chart)
    - [Installing Khulnasoft Scanner from Helm Private Repository](#installing-khulnasoft-scanner-from-helm-private-repository)
    - [(Optional) Configure SSL communication with Khulnasoft Server](#optional-configure-ssl-communication-with-khulnasoft-server)
    - [(Optional) Configure mTLS communication with offline Khulnasoft CyberCenter in air-gap environment](#optional-configure-mtls-communication-with-offline-khulnasoft-cybercenter-in-air-gap-environment)
  - [Configurable Variables](#configurable-variables)
    - [Scanner](#scanner)
  - [Issues and feedback](#issues-and-feedback)

## Prerequisites

### Container Registry Credentials

[Link](../docs/imagepullsecret.md)

### Scanner User Credentials

Before installing scanner chart the recommendation is to create user with scanning permissions, [Link to documentations](https://docs.khulnasoft.com/docs/add-scanners#section-add-a-scanner-user)

## Installing the Chart
Follow the steps in this section for production grade deployments. You can either clone khulnasoft-helm git repo or you can add our helm private repository ([https://helm.khulnasoft.com](https://helm.khulnasoft.com))

### Installing Khulnasoft Scanner from Helm Private Repository

* Add Khulnasoft Helm Repository
```shell
helm repo add khulnasoft-helm https://helm.khulnasoft.com
helm repo update
```

* Check for available chart versions either from [Changelog](./CHANGELOG.md) or by running the below command
```shell
helm search repo khulnasoft-helm/scanner --versions
```

* Install Khulnasoft

```shell
helm upgrade --install --namespace khulnasoft scanner khulnasoft-helm/scanner --set imageCredentials.username=<>,imageCredentials.password=<>,user=<>,password=<>,scannerToken=<>
```
### (Optional) Configure SSL communication with Khulnasoft Server
To enable SSL communication from the Scanner to the Khulnasoft Server. Perform these steps:
1. enable `serverSSL.enabled` to true in values.yaml or use `--set 'serverSSL.enabled=true'` for inline argument
2. You need to encode the certificate into base64 for ```server.crt```  using this command:

    ```bash
    cat <file-name> | base64 | tr -d '\n'
    ```
3. Provide the certificates previously obtained in the fields of the `serverSSL.serverSSLCert` in values.yaml or use `--set serverSSL.serverSSLCert=<cert base64 value>`

### (Optional) Configure mTLS communication with offline Khulnasoft CyberCenter in air-gap environment
1. To establish mTLS with offline cyberCenter, create secret using the below command ``` kubectl create secret generic khulnasoft-grpc-scanner --from-file=<rootCA.crt> --from-file=<khulnasoft_scanner.crt> --from-file=<khulnasoft_scanner.key> -n khulnasoft```
2. Enable `cyberCenter.mtls.enabled` to true in `values.yaml`
3. Add previously created secret name `cyberCenter.mtls.secretName` in `values.yaml`
4. Add respective certificate file names to `cyberCenter.mtls.publicKey_fileName`, `cyberCenter.mtls.privateKey_fileName`, `cyberCenter.mtls.rootCA_fileName`(Add rootCA if certs are self-signed)

`KHULNASOFT_PRIVATE_KEY`, `KHULNASOFT_PUBLIC_KEY`, `KHULNASOFT_ROOT_CA`(if using self-signed certs), `OFFLINE_CC_MTLS_ENABLE` in [scanner_configmap.yaml](./templates/scanner-configmap.yaml) and uncomment khulnasoft-grpc-scanner certs volume mount and volumes block from [scanner-deployment.yaml](./templates/scanner-deployment.yaml) file before applying the kubectl commands.

Before installing scanner chart the recommendation is to create user with scanning permissions, [Link to documentations](https://docs.khulnasoft.com/docs/add-scanners#section-add-a-scanner-user)

## Configurable Variables

The following table lists the configurable parameters of the Console and Enforcer charts with their default values.

### Scanner

| Parameter                              | Description                                                                                                                                                  | Default                | Mandatory                                                                 |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------|---------------------------------------------------------------------------|
| `imageCredentials.create`              | Enables to create image credentials                                                                                                                          | `false`                | `NO`                                                                      |
| `imageCredentials.name`                | If `imageCredentials.create` is enabled sets imageCredentials name                                                                                           |                        | `NO`                                                                      |
| `imageCredentials.registry`            | The registry that the credentials will be created for. mandatory if `imageCredentials.create` is enabled                                                     | `registry.khulnasoft.com` | `NO`                                                                      |
| `imageCredentials.username`            | The username for the registry mandatory if `imageCredentials.create` is enabled                                                                              |                        | `NO`                                                                      |
| `imageCredentials.password`            | The password for the registry mandatory if `imageCredentials.create` is enabled                                                                              |                        | `NO`                                                                      |
| `dockerSocket.mount`                   | Boolean parameter if to mount docker socket                                                                                                                  | `unset`                | `NO`                                                                      |
| `dockerSocket.path`                    | Docker socket path                                                                                                                                           | `/var/run/docker.sock` | `NO`                                                                      |
| `platform`                             | Specify the Kubernetes (k8s) platform acronym, allowed values are: aks, eks, gke, openshift, tkg, tkgi, k8s, rancher, gs, k3s, mke.                          | `unset`                | `YES`                                                                     |
| `directCC.enabled`                     | Scanner talk to the cyberCenter directly                                                                                                                     | `yes`                  | `NO`                                                                      |
| `serviceAccount.create`                | Enable to create serviceAccount if not exist in the Kubernetes                                                                                               | `false`                | `NO`                                                                      |
| `serviceAccount.name`                  | Kubernetes service-account name either existing one or new name if create is enabled                                                                         | `khulnasoft-sa`              | `YES`                                                                     |
| `server.scheme`                        | Scheme for server to connect                                                                                                                                 | `http`                 | `NO`                                                                      |
| `server.serviceName`                   | Service name for server to connect                                                                                                                           | `khulnasoft-console-svc`     | `YES`                                                                     |
| `server.port`                          | Service port for server to connect                                                                                                                           | `8080`                 | `YES`                                                                     |
| `serverSSl.enable`                     | To establish SSL communication with Khulnasoft Server                                                                                                              | `false`                | `NO`                                                                      |
| `serverSSl.createSecret`               | Change to false if you're using existing server certificate secret                                                                                           | `true`                 | `YES` <br /> `if serverSSl.enable is set to true `                        |
| `serverSSL.secretName`                 | Secret name for the SSL cert                                                                                                                                 | `scanner-web-cert`     | `YES` <br /> `if serverSSl.enable is set to true `                        |
| `serverSSL.cert_file`                  | If serverSSL createSecret enable to true, add base64 value of the the server public certificate or add filename of certificate if loading from custom secret | `Nill`                 | `YES` <br /> `if serverSSl.enable is set to true `                        |
| `cyberCenter.mtls.enabled`             | If require secure channel communication                                                                                                                      | `false`                | `NO`                                                                      |
| `cyberCenter.mtls.secretName`          | Certificates secret name                                                                                                                                     | `nil`                  | `NO`                                                                      |
| `cyberCenter.mtls.publicKey_fileName`  | Filename of the public key eg: khulnasoft_scanner.crt                                                                                                              | `nil`                  | `YES` <br /> `if cyberCenter.mtls.enabled is set to true`                 |
| `cyberCenter.mtls.privateKey_fileName` | Filename of the private key eg: khulnasoft_scanner.key                                                                                                             | `nil`                  | `YES` <br /> `if cyberCenter.mtls.enabled is set to true`                 |
| `cyberCenter.mtls.rootCA_fileName`     | Filename of the rootCA, if using self-signed certificates eg: rootCA.crt                                                                                     | `nil`                  | `NO` <br /> `if using self-signed certificates for mTLS`                  |
| `image.repository`                     | The docker image name to use                                                                                                                                 | `scanner`              | `YES`                                                                     |
| `image.tag`                            | The image tag to use.                                                                                                                                        | `2022.4`               | `YES`                                                                     |
| `image.pullPolicy`                     | The kubernetes image pull policy.                                                                                                                            | `IfNotPresent`         | `NO`                                                                      |
| `user`                                 | Scanner username                                                                                                                                             | `unset`                | `YES`                                                                     |
| `password`                             | Scanner password                                                                                                                                             | `unset`                | `YES`                                                                     |
| `nameOverride`                         | Scanner name                                                                                                                                                 | `khulnasoft-scanner`         | `NO`                                                                      |
| `scannerUserSecret.enable`             | Change it to true for loading scanner user, scanner password from secret                                                                                     | `false`                | `YES` <br /> `If password is not declared`                                |
| `scannerUserSecret.secretName`         | Secret name for the scanner user, scanner password secret                                                                                                    | `null`                 | `YES` <br /> `If password is not declared`                                |
| `scannerUserSecret.userKey`            | Secret key of the scanner user                                                                                                                               | `null`                 | `YES` <br /> `If password is not declared`                                |
| `scannerUserSecret.passwordKey`        | Secret key of the scanner password                                                                                                                           | `null`                 | `YES` <br /> `If password is not declared`                                |
| `scannerTokenSecret.enable`            | Change it to true for loading the scanner token from secrets                                                                                                 | `false`                | `YES` <br /> `If scannerToken and username and password are not declared` |
| `scannerTokenSecret.secretName`        | Secret name for the scanner token secret                                                                                                                     | `null`                 | `YES` <br /> `If scannerToken and username and password are not declared` |
| `scannerTokenSecret.tokenKey`          | Scanner token key name                                                                                                                                       | `null`                 | `YES` <br /> `If scannerToken and username and password are not declared` |
| `replicaCount`                         | Replica count                                                                                                                                                | `1`                    | `NO`                                                                      |
| `resources`                            | Resource requests and limits                                                                                                                                 | `{}`                   | `NO`                                                                      |
| `enableLivenessProbe`                  | LivenessProbe                                                                                                                                                | `false`                | `true`                                                                    |
| `nodeSelector`                         | Kubernetes node selector	                                                                                                                                    | `{}`                   | `NO`                                                                      |
| `tolerations`                          | Kubernetes node tolerations	                                                                                                                                 | `[]`                   | `NO`                                                                      |
| `affinity`                             | Kubernetes node affinity                                                                                                                                     | `{}`                   | `NO`                                                                      |
| `podAnnotations`                       | Kubernetes pod annotations                                                                                                                                   | `{}`                   | `NO`                                                                      |
| `extraEnvironmentVars`                 | Is a list of extra environment variables to set in the scanner deployments.                                                                                  | `{}`                   | `NO`                                                                      |
| `extraSecretEnvironmentVars`           | Is a list of extra environment variables to set in the scanner deployments, these variables take value from existing Secret objects.                         | `[]`                   | `NO`                                                                      |
| `rbac.create`                          | To create default cluster-role and cluster-role bindings according to the platform                                                                           | `true`                 | `NO`                                                                      |
| `clusterRole.roleRef`                  | Cluster role reference name for cluster roleBinding                                                                                                          | `unset`                | `NO`                                                                      |
| `extraVolumes`                         | extraVolumes is a list of volumes that can be mounted inside the KubeEnforcer deployment                                                                     | `[]`                   | `NO`                                                                      |
| `extraVolumeMounts`                    | extraVolumeMounts is a list of extra volumes to mount into the container's filesystem of the KubeEnforcer deployment                                         | `[]`                   | `NO`                                                                      |

## Issues and feedback

If you encounter any problems or would like to give us feedback on deployments, we encourage you to raise issues here on GitHub.
