<img src="https://avatars3.githubusercontent.com/u/43526139?s=200&v=4" height="100" width="100" /><img src="https://avatars3.githubusercontent.com/u/15859888?s=200&v=4" width="100" height="100"/>

# KhulnaSoft Security Tenant Manager Helm Chart

These are Helm charts for the deployment and maintenance of the Khulnasoft Tenant Manager.
## Contents

- [KhulnaSoft Security Tenant Manager Helm Chart](#khulnasoft-security-tenant-manager-helm-chart)
  - [Contents](#contents)
  - [Prerequisites](#prerequisites)
    - [Container registry credentials](#container-registry-credentials)
    - [PostgreSQL database](#postgresql-database)
  - [Installing the Chart](#installing-the-chart)
    - [Installing Khulnasoft Tenant Manager from GitHub Repo](#installing-khulnasoft-tenant-manager-from-github-repo)
    - [Installing Khulnasoft Tenant Manager from Helm Private Repository](#installing-khulnasoft-tenant-manager-from-helm-private-repository)
  - [Database](#database)
  - [Configuring HTTPS for the Khulnasoft Tenant Manager](#configuring-https-for-the-khulnasoft-tenant-manager)
  - [Configurable variables](#configurable-variables)
  - [Issues and feedback](#issues-and-feedback)

## Prerequisites

### Container registry credentials

See [Container registry credentials](../docs/imagepullsecret.md).

### PostgreSQL database

KhulnaSoft Security recommends implementing a highly-available PostgreSQL database. By default, the Tenant Manager chart will install a PostgreSQL database and attach it to persistent storage for POC usage and testing. For production use, you can override this default behavior and specify an existing PostgreSQL database by setting the following variables in values.yaml:

```yaml
db:
  external:
    enabled: true
    name: example-khulnasoft
    host: khulnasoft-db
    port: 5432
    user: khulnasoft-db-username
    password: verysecret
```
## Installing the Chart
Follow the steps in this section for production-grade deployments. You can either clone the khulnasoft-helm GitHub repo or you can add our private Helm repository ([https://helm.khulnasoft.com](https://helm.khulnasoft.com)).

### Installing Khulnasoft Tenant Manager from Helm Private Repository
  
* Add the Khulnasoft Helm repository

```shell
helm repo add khulnasoft-helm https://helm.khulnasoft.com
```

* Check for available chart versions either from [Changelog](./CHANGELOG.md) or by running the below command
```shell
helm search repo khulnasoft-helm/tenant-manager --versions
```

* Deploy the Khulnasoft Tenant Manager

```shell
helm upgrade --install --namespace khulnasoft tenant-manager khulnasoft-helm/tenant-manager --set imageCredentials.username=<>,imageCredentials.password=<>,platform=<>
```

## Database

   1. By default, the Khulnasoft Helm chart will deploy a database container. If you wish to use an external database, set `db.external.enabled` to true and the following with appropriate values:
      ```shell
      1. db.external.name
      2. db.external.host
      3. db.external.port
      4. db.external.user
      5. db.external.password
      ```
   2. By default, the same database (Packaged DB Container | Managed DB like AWS RDS) will be used to host both the main DB and the audit DB. If you want to use a different database for the audit DB, set the following variables in the values.yaml file:
      ```shell
      1. db.external.auditName
      2. db.external.auditHost
      3. db.external.auditPort
      4. db.external.auditUser
      5. db.external.auditPassword      
      ```
   3. If you are using the Khulnasoft packaged DB container then:
   
      * The `KHULNASOFT_ENV_SIZE` variable can be used to define the size of your DB container in terms of the number of connections and optimized configuration (but not the PV size). Choose the appropriate PV size as per your requirements.
      * By default, `KHULNASOFT_ENV_SIZE` is set to `"S"`; other allowed values are `"M"` and `"L"`.
   
## Configuring HTTPS for the Khulnasoft Tenant Manager

   By default, Khulnasoft will generate a self-signed cert and will use the same for HTTPS communication. If you wish to use your own SSL/TLS certs, you can do this in two different ways:

   e.g., LoadBalancer(Default): Use the `KHULNASOFT_PUBLIC_KEY`, `KHULNASOFT_PRIVATE_KEY`, and `KHULNASOFT_ROOT_CA` environment variables to specify the TLS cert path. Make sure to mount the TLS cert into the container.


## Configurable variables

| Parameter                                   | Description                                                                                                                             | Default                | Mandatory                                          |
|---------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|------------------------|----------------------------------------------------|
| `imageCredentials.create`                   | Whether to create a new pull image secret                                                                                               | `true`                 | `YES`                                              |
| `imageCredentials.name`                     | Your Docker pull image secret name                                                                                                      | `khulnasoft-registry-secret` | `YES`                                              |
| `imageCredentials.repositoryUriPrefix`      | Repository URI prefix for Docker Hub set `docker.io`                                                                                    | `registry.khulnasoft.com` | `YES`                                              |
| `imageCredentials.registry`                 | Registry URL for Docker Hub `index.docker.io/v1/`                                                                                       | `registry.khulnasoft.com` | `YES`                                              |
| `imageCredentials.username`                 | Your Docker registry (Docker Hub, etc.) username                                                                                        | `khulnasoft-registry-secret` | `YES`                                              |
| `imageCredentials.password`                 | Your Docker registry (Docker Hub, etc.) password                                                                                        | `unset`                | `YES`                                              |
| `platform`                                  | Specify the Kubernetes (k8s) platform acronym, allowed values are: aks, eks, gke, openshift, tkg, tkgi, k8s, rancher, gs, k3s, mke.     | `unset`                | `YES`                                              |
| `clusterRole.roleRef`                       | Cluster role reference name for cluster rolebinding                                                                                     | `unset`                | `NO`                                               |
| `admin.token`                               | Use this Khulnasoft license token                                                                                                             | `unset`                | `NO`                                               |
| `admin.password`                            | Use this Khulnasoft admin password                                                                                                            | `unset`                | `NO`                                               |
| `db.external.enabled`                       | Avoid installing the packaged DB (Postgres container); use an external database instead                                                 | `false`                | `YES`                                              |
| `db.external.name`                          | PostgreSQL DB name                                                                                                                      | `unset`                | `YES`<br />`if db.external.enabled is set to true` |
| `db.external.host`                          | PostgreSQL DB hostname                                                                                                                  | `unset`                | `YES`<br />`if db.external.enabled is set to true` |
| `db.external.port`                          | PostgreSQL DB port                                                                                                                      | `unset`                | `YES`<br />`if db.external.enabled is set to true` |
| `db.external.user`                          | PostgreSQL DB username                                                                                                                  | `unset`                | `YES`<br />`if db.external.enabled is set to true` |
| `db.external.password`                      | PostgreSQL DB password                                                                                                                  | `unset`                | `YES`<br />`if db.external.enabled is set to true` |
| `db.external.auditName`                     | PostgreSQL DB audit name                                                                                                                | `unset`                | `NO`                                               |
| `db.external.auditHost`                     | PostgreSQL DB audit hostname                                                                                                            | `unset`                | `NO`                                               |
| `db.external.auditPort`                     | PostgreSQL DB audit port                                                                                                                | `unset`                | `NO`                                               |
| `db.external.auditUser`                     | PostgreSQL DB audit username                                                                                                            | `unset`                | `NO`                                               |
| `db.external.auditPassword`                 | PostgreSQL DB audit password                                                                                                            | `unset`                | `NO`                                               |
| `db.passwordFromSecret.enabled`             | Enable to load DB passwords from ecrets                                                                                                 | `false`                | `YES`                                              |
| `db.passwordFromSecret.dbPasswordName`      | Password secret name                                                                                                                    | `null`                 | `NO`                                               |
| `db.passwordFromSecret.dbPasswordKey`       | Password secret key                                                                                                                     | `null`                 | `NO`                                               |
| `db.passwordFromSecret.dbAuditPasswordName` | Audit password secret name                                                                                                              | `null`                 | `NO`                                               |
| `db.passwordFromSecret.dbAuditPasswordKey`  | Audit password secret key                                                                                                               | `null`                 | `NO`                                               |
| `db.persistence.enabled`                    | If true, a PVC (persistent volume claim) will be created                                                                                | 	`true`                | `NO`                                               |
| `db.persistence.accessModes`                | Persistent volume access mode                                                                                                           | 	`ReadWriteOnce`       | `NO`                                               |
| `db.persistence.size`                       | Persistent volume size                                                                                                                  | `30Gi`                 | `NO`                                               |
| `db.persistence.storageClass`               | Persistent volume storage class                                                                                                         | `unset`                | `NO`                                               |
| `db.image.repository`                       | Docker image name to use                                                                                                                | `database`             | `NO`                                               |
| `db.image.tag`                              | Image tag to use                                                                                                                        | `2022.4`               | `NO`                                               |
| `db.image.pullPolicy`                       | Kubernetes image pull policy                                                                                                            | `IfNotPresent`         | `NO`                                               |
| `db.service.type`                           | Kubernetes service type                                                                                                                 | `ClusterIP`            | `NO`                                               |
| `db.resources`                              | Resource requests and limits                                                                                                            | `{}`                   | `NO`                                               |
| `db.nodeSelector`                           | Kubernetes node selector	                                                                                                               | `{}`                   | `NO`                                               |
| `db.tolerations`                            | Kubernetes node tolerations	                                                                                                            | `[]`                   | `NO`                                               |
| `db.affinity`                               | Kubernetes node affinity                                                                                                                | `{}`                   | `NO`                                               |
| `db.podAnnotations`                         | Kubernetes pod annotations                                                                                                              | `{}`                   | `NO`                                               |
| `db.securityContext`                        | Set of security context for the container                                                                                               | `nil`                  | `NO`                                               |
| `db.extraEnvironmentVars`                   | List of extra environment variables to set in the database deployments                                                                  | `{}`                   | `NO`                                               |
| `db.extraSecretEnvironmentVars`             | List of extra environment variables to set in the database deployments; these variables take values from existing Secret objects        | `[]`                   | `NO`                                               |
| `tenantmanager.image.repository`            | Docker image name to use                                                                                                                | `tenantmanager`        | `NO`                                               |
| `tenantmanager.image.tag`                   | Image tag to use                                                                                                                        | `2022.4`               | `NO`                                               |
| `tenantmanager.image.pullPolicy`            | Kubernetes image pull policy                                                                                                            | `IfNotPresent`         | `NO`                                               |
| `tenantmanager.service.type`                | Kubernetes service type                                                                                                                 | `LoadBalancer`         | `NO`                                               |
| `tenantmanager.service.annotations`         | Service annotations	                                                                                                                    | `{}`                   | `NO`                                               |
| `tenantmanager.service.ports`               | Array of port settings                                                                                                                  | `array`                | `NO`                                               |
| `tenantmanager.replicaCount`                | Replica count                                                                                                                           | `1`                    | `NO`                                               |
| `tenantmanager.resources`                   | Resource requests and limits                                                                                                            | `{}`                   | `NO`                                               |
| `tenantmanager.nodeSelector`                | Kubernetes node selector	                                                                                                               | `{}`                   | `NO`                                               |
| `tenantmanager.tolerations`                 | Kubernetes node tolerations	                                                                                                            | `[]`                   | `NO`                                               |
| `tenantmanager.affinity`                    | Kubernetes node affinity                                                                                                                | `{}`                   | `NO`                                               |
| `tenantmanager.podAnnotations`              | Kubernetes pod annotations                                                                                                              | `{}`                   | `NO`                                               |
| `tenantmanager.ingress.enabled`             | If true, Ingress will be created                                                                                                        | `false`                | `NO`                                               |
| `tenantmanager.ingress.annotations`         | Ingress annotations	                                                                                                                    | `[]`                   | `NO`                                               |
| `tenantmanager.ingress.hosts`               | Ingress hostnames                                                                                                                       | 	`[]`                  | `NO`                                               |
| `tenantmanager.ingress.tls`                 | Ingress TLS configuration (YAML)                                                                                                        | `[]`                   | `NO`                                               |
| `tenantmanager.securityContext`             | Set of security context for the container                                                                                               | `nil`                  | `NO`                                               |
| `tenantmanager.TLS.enabled`                 | Whether to require secure channel communication                                                                                         | `false`                | `NO`                                               |
| `tenantmanager.TLS.secretName`              | Certificates secret name                                                                                                                | `nil`                  | `NO`                                               |
| `tenantmanager.maintenance_db.name`         | If Configured to use custom maintenance DB specify the DB name                                                                          | `nil`                  | `NO`                                               |
| `tenantmanager.extraEnvironmentVars`        | List of extra environment variables to set in the Tenant Manager deployments                                                            | `{}`                   | `NO`                                               |
| `tenantmanager.extraSecretEnvironmentVars`  | List of extra environment variables to set in the Tenant Manager deployments; these variables take values from existing Secret objects. | `[]`                   | `NO`                                               |

## Issues and feedback

If you encounter any problems or would like to give us feedback on deployments, we encourage you to raise issues here on GitHub.
