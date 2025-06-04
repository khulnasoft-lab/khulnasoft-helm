<img src="https://avatars3.githubusercontent.com/u/43526139?s=200&v=4" height="100" width="100" /><img src="https://avatars3.githubusercontent.com/u/15859888?s=200&v=4" width="100" height="100"/>

# Khulnasoft Security CyberCenter Helm Chart

These are Helm charts for installing and maintaining the Khulnasoft Security CyberCenter.

## Contents

- [Khulnasoft Security CyberCenter Helm Chart](#khulnasoft-security-cybercenter-helm-chart)
  - [Contents](#contents)
  - [Prerequisites](#prerequisites)
    - [Container Registry Credentials](#container-registry-credentials)
  - [Installing the Chart](#installing-the-chart)
    - [Installing Khulnasoft CyberCenter from Helm Private Repository](#installing-khulnasoft-cybercenter-from-helm-private-repository)
  - [Configuring mTLS/TLS](#configuring-mtlstls)
  - [How to connect to the offline CyberCenter from the Khulnasoft console](#how-to-connect-to-the-offline-cybercenter-from-the-khulnasoft-console)
  - [Configurable Variables for the CyberCenter](#configurable-variables-for-the-cybercenter)
  - [Issues and feedback](#issues-and-feedback)

## Prerequisites

### Container Registry Credentials

[Link](../docs/imagepullsecret.md)

## Installing the Chart
Follow the steps in this section for production grade deployments. You can either clone khulnasoft-helm git repo or you can add our helm private repository ([https://helm.khulnasoft.com](https://helm.khulnasoft.com))

### Installing Khulnasoft CyberCenter from Helm Private Repository

* Add Khulnasoft Helm Repository
```shell
helm repo add khulnasoft-helm https://helm.khulnasoft.com
helm repo update
```

* Check for available chart versions either from [Changelog](./CHANGELOG.md) or by running the following command:
```shell
helm search repo khulnasoft-helm/cyber-center --versions
```

* Install Khulnasoft CyberCenter

```shell
helm upgrade --install --namespace khulnasoft khulnasoft-cyber-center khulnasoft-helm/cyber-center --set imageCredentials.username=<>,imageCredentials.password=<>
```

## Configuring mTLS/TLS

In order to support L7 / gRPC communication between the CyberCenter and the Server, or between the CyberCenter and scanners, it is recommended to follow the detailed steps to deploy and enable the CyberCenter.

   1. The CyberCenter should connect to the Envoy/Gateway service and verify its certificate. If Envoy/Gateway certificate is signed by a public provider (e.g., Let’s Encrypt), the CyberCenter will be able to verify the certificate without being given a root certificate. Otherwise, the Envoy/Gateway root certificate should be accessible to the CyberCenter from an environment variable path that is mounted at /opt/khulnasoft/ssl/.

      ```shell
      # Self-Signed Root CA (Optional)
      #####################################################################################
      # Create Root Key
      # If you want a non password protected key just remove the -des3 option
      openssl genrsa -des3 -out rootCA.key 4096
      # Create and self sign the Root Certificate
      openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
      #####################################################################################
      # Create a certificate
      #####################################################################################
      # Create the certificate key
      openssl genrsa -out khulnasoft_cc_mydomain.com.key 2048
      # Create the signing (csr)
      openssl req -new -key khulnasoft_cc_mydomain.com.key -out khulnasoft_cc_mydomain.com.csr
      # Verify the csr content
      openssl req -in khulnasoft_cc_mydomain.com.csr -noout -text
      #####################################################################################
      # Generate the certificate using the khulnasoft_cc_mydomain csr and key along with the CA Root key
      #####################################################################################
      openssl x509 -req -in khulnasoft_cc_mydomain.com.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out khulnasoft_cc_mydomain.com.crt -days 500 -sha256
      #####################################################################################

      # If you wish to use a Public CA like GoDaddy or Let’s Encrypt please
      # submit the khulnasoft_cc_mydomain csr to the respective CA to generate khulnasoft_cc_mydomain crt
      ```

   2. Create CyberCenter agent cert secret

      ```shell
      ## Example: 
      ## Change < certificate filenames > respectively
      kubectl create secret generic khulnasoft-cc-certs --from-file <khulnasoft_cyber-center_private.key> --from-file <khulnasoft_cyber-center_public.crt> --from-file <rootCA.crt> -n khulnasoft
      ```

   3. Enable `TLS.enable`  to `true` in values.yaml
   4. Add the certificate's secret name `TLS.secretName` in values.yaml
   5. Add the respective certificate file names to `TLS.publicKey_fileName`, `TLS.privateKey_fileName` and `TLS.rootCA_fileName`(Add rootCA if certs are self-signed) in values.yaml

## How to connect to the offline CyberCenter from the Khulnasoft console

Log in into the Khulnasoft web UI then go to the `Khulnasoft CyberCenter` section under the `Settings` tab to connect to a new offline CyberCenter. 

1. Change the address to the offline CyberCenter address, e.g., `http://khulnasoft-cc:443`.
2. Test the connection; once it is successful, save the changes.
3. Disable `Fast Scanning` under the `Scanning` section in the `Settings` tab.

For more information, refer to [Link](https://docs.khulnasoft.com/docs/cybercenter-configuration)

## Configurable Variables for the CyberCenter

| Parameter                              | Description                                                                 | Default                  | Mandatory                                                                                     |
|----------------------------------------|-----------------------------------------------------------------------------|--------------------------|-----------------------------------------------------------------------------------------------|
| `imageCredentials.create`              | Enable to create new pull image secret                                      | `false`                  | `YES - New cluster`                                                                           |
| `imageCredentials.name`                | Your Docker pull image secret name                                          | `khulnasoft-registry-secret`   | `YES - New cluster`                                                                           |
| `imageCredentials.repositoryUriPrefix` | Repository URI prefix for Docker Hub set `docker.io`                        | `registry.khulnasoft.com`   | `YES - New cluster`                                                                           |
| `imageCredentials.registry`            | Set the registry URL for Docker Hub set `index.docker.io/v1/`               | `registry.khulnasoft.com`   | `YES - New cluster`                                                                           |
| `imageCredentials.username`            | Your Docker registry (Docker Hub, etc.) username                            | `khulnasoft-registry-secret`   | `YES - New cluster`                                                                           |
| `imageCredentials.password`            | Your Docker registry (Docker Hub, etc.) password                            | `unset`                  | `YES - New cluster`                                                                           |
| `serviceAccount.name`                  | Service account name                                                        | `khulnasoft-sa`                | `NO`                                                                                          |
| `serviceAccount.create`                | Enable to create khulnasoft-sa serviceAccount if it is missing in the environment | `false`                  | `YES - New cluster`                                                                           |
| `platform`                             | Platform name (for Openshift)                                               | ``                       | `NO`                                                                                          |
| `image.repository`                     | Docker image name to use                                                    | `cc-premium`             | `YES`                                                                                         |
| `image.tag`                            | Image tag to use                                                            | `2022.4`                 | `YES`                                                                                         |
| `image.pullPolicy`                     | Kubernetes image pull policy                                                | `Always`                 | `NO`                                                                                          |
| `service.type`                         | Kubernetes service type                                                     | `ClusterIP`              | `NO`                                                                                          |
| `service.annotations`                  | Service annotations	                                                        | `{}`                     | `NO`                                                                                          |
| `service.ports`                        | Array of ports settings                                                     | `array`                  | `NO`                                                                                          |
| `tolerations`                          | Kubernetes node tolerations	                                                | `[]`                     | `NO`                                                                                          |
| `deploymentAnnotations`                | Kubernetes deployment annotations                                           | `{}`                     | `NO`                                                                                          |
| `podAnnotations`                       | Kubernetes pod annotations                                                  | `{}`                     | `NO`                                                                                          |
| `resources`                            | Resource requests and limits                                                | `{}`                     | `NO`                                                                                          |
| `nodeSelector`                         | Kubernetes node selector	                                                | `{}`                     | `NO`                                                                                          |
| `affinity`                             | Kubernetes node affinity                                                    | `{}`                     | `NO`                                                                                          |
| `TLS.enabled`                          | If secure channel communication is required                                 | `false`                  | `NO`                                                                                          |
| `TLS.secretName`                       | Certificates secret name                                                    | `nil`                    | `YES` <br /> `if TLS.enabled is set to true`                                                  |
| `TLS.publicKey_fileName`               | Filename of the public key, e.g., khulnasoft_cyber-center.crt                     | `nil`                    | `YES` <br /> `if TLS.enabled is set to true`                                                  |
| `TLS.privateKey_fileName`              | Filename of the private key, e.g., khulnasoft_cyber-center.key                    | `nil`                    | `YES` <br /> `if TLS.enabled is set to true`                                                  |
| `TLS.rootCA_fileName`                  | Filename of the rootCA, if using self-signed certificates, e.g., rootCA.crt | `nil`                    | `NO` <br /> `if TLS.enabled is set to true and using self-signed certificates for TLS/mTLS`   |
|----------------------------------------|-----------------------------------------------------------------------------| ------------------------ |-----------------------------------------------------------------------------------------------|
> Note: `imageCredentials.create` is false; if you need to create an image pull secret, update this to true, set the username and password for the registry, and set `serviceAccount.create` to false. If your environment is new or not having khulnasoft-sa serviceAccount, update it to true.

## Issues and feedback

If you encounter any problems or would like to give us feedback on deployments, we encourage you to raise issues here on GitHub.
