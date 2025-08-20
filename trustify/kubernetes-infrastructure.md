---
title: Preparing the Infrastructure
layout: page
permalink: /trustify/infrastructure/
nav_order: 1
parent: Kubernetes
---

# Infrastructure

Trustify requires some infrastructure services for its installation. The
services required are:

- OIDC provider
- PostgreSQL database instance
- Storage, either:
  - Filesystem
  - S3 compatible

Those services have to be provided by the user before the installation is being
performed. Some information, like access credentials, must be provided during
the installation of Trustify.

There are different ways to set up such services. However, it is up to the user
to provide those services and set them up.

The following sections provide a few examples on how they can be provided in
different ways. Keep in mind, those are just examples, and you can modify them
to suit your needs, or choose a different approach in providing those services.

## Self-managed Kubernetes

A simple approach is to use Keycloak as an OIDC provider, a PostgreSQL
container, and a persistent volume claim for the filesystem storage.

To set this up, it is possible to just use existing Helm charts for Keycloak and
PostgreSQL. We do provide an opinionated infrastructure Helm chart for this case
at:
https://github.com/trustification/trustify-helm-charts/tree/main/charts/trustify-infrastructure

You can install this using:

```
NAMESPACE=trustify
APP_DOMAIN=public.cluster.domain
helm upgrade --install -n $NAMESPACE --repo https://trustification.io/trustify-helm-charts/ infrastructure trustify-infrastructure --values <values-file> --set-string keycloak.ingress.hostname=sso$APP_DOMAIN --set-string appDomain=$APP_DOMAIN
```

For this, you will need to provide a Helm "values" file. Which is a YAML file,
providing additional information for the Helm chart.

An example file, for Minikube, is:

```
keycloak:
  enabled: true
  production: false
  auth:
    adminUser: admin
    adminPassword: admin123456 # notsecret, replace
  tls:
    enabled: false
  service: {}
  ingress:
    enabled: true
    servicePort: http

oidc:
  clients:
    frontend: {}
    cli:
      clientSecret:
        value: 5460cc91-4e20-4edd-881c-b15b169f8a79 # notsecret, replace
```

## AWS services

It also is possible to use AWS managed services. The following AWS services can
be used:

- OIDC provider: AWS Cognito
- PostgreSQL database instance: AWS RDS
- Storage: AWS S3

### Manual setup

You can create the AWS resources manually, either through the AWS console or
using the AWS CLI.

### Terraform with OpenShift

Trustify also provides an example Terraform setup, which is intended to quickly
deploy an opinionated set of services. The Terraform scripts will create the AWS
resources, as well as create Kubernetes resources with information from the
Terraform creation process, so that the Helm charts can pick up this
information.

#### Main module

To use the Terraform scripts, you will need to create a wrapper/main module,
referencing this trustify module.

{: .note } The following example file needs to be adapted to your needs. Example
values have to be replaced with values that suit your deployment.

```
provider "aws" {
  region  = "<your region>"
  profile = "<your aws cli profile>"
}

provider "kubernetes" {
  config_path    = "<path to kubeconfig>"
  config_context = "<name of the kubectl context>"
}

variable "app-domain" {
  type = string
}

module "trustify" {
  source = "./trustify"

  cluster-vpc-id = "<your cluster vpc>"
  availability-zone = "<your availability zone inside your region>"

  namespace = "trustify"

  admin-email = "<your e-mail address>"
  sso-domain = "<a free cognito console domain name>"
  console-url = "https://server${var.app-domain}"
}
```

#### Creating the resources

First, initialize the OpenTofu instance. This will set up the required providers
and does not yet create any resources:

`tofu init`

The following commands require the environment variable `APP_DOMAIN` to be set.
You can do this using the following command:

```
NAMESPACE=trustify
APP_DOMAIN=-$NAMESPACE.$(kubectl -n openshift-ingress-operator get ingresscontrollers.operator.openshift.io default -o jsonpath='{.status.domain}')
```

Then, check if the resources can be created. This does not yet create the
resources:

`tofu plan --var app-domain=$APP_DOMAIN`

This will show you the resources which will get created and check if the
creation is expected to be successful.

If this worked fine, proceed with actually creating the resources:

`tofu apply --var app-domain=$APP_DOMAIN`

This will also create some resources in the Kubernetes cluster, including the
credentials to the AWS accounts created for accessing the created AWS resources.

### Running the Helm chart

Prepare a "values" files, named `values-ocp-aws.yaml`:

```
ingress:
  className: openshift-default

authenticator:
  type: cognito

storage:
  type: s3
  region:
    valueFrom:
      configMapKeyRef:
        name: aws-storage
        key: region
  bucket: trustify
  accessKey:
    valueFrom:
      secretKeyRef:
        name: storage-credentials
        key: aws_access_key_id
  secretKey:
    valueFrom:
      secretKeyRef:
        name: storage-credentials
        key: aws_secret_access_key

database:
  host:
    valueFrom:
      secretKeyRef:
        name: postgresql-credentials
        key: db.host
  port:
    valueFrom:
      secretKeyRef:
        name: postgresql-credentials
        key: db.port
  name:
    valueFrom:
      secretKeyRef:
        name: postgresql-credentials
        key: db.name
  username:
    valueFrom:
      secretKeyRef:
        name: postgresql-credentials
        key: db.user
  password:
    valueFrom:
      secretKeyRef:
        name: postgresql-credentials
        key: db.port

createDatabase:
  name:
    valueFrom:
      secretKeyRef:
        name: postgresql-admin-credentials
        key: db.name
  username:
    valueFrom:
      secretKeyRef:
        name: postgresql-admin-credentials
        key: db.user
  password:
    valueFrom:
      secretKeyRef:
        name: postgresql-admin-credentials
        key: db.password

migrateDatabase:
  username:
    valueFrom:
      secretKeyRef:
        name: postgresql-admin-credentials
        key: db.user
  password:
    valueFrom:
      secretKeyRef:
        name: postgresql-admin-credentials
        key: db.password

modules:
  createDatabase:
    enabled: true
  migrateDatabase:
    enabled: true

oidc:
  issuerUrl:
    valueFrom:
      configMapKeyRef:
        name: aws-oidc
        key: issuer-url
  clients:
    frontend:
      clientId:
        valueFrom:
          secretKeyRef:
            name: oidc-frontend
            key: client-id
    cli:
      clientId:
        valueFrom:
          secretKeyRef:
            name: oidc-cli
            key: client-id
      clientSecret:
        valueFrom:
          secretKeyRef:
            name: oidc-cli
            key: client-secret
```

You can now run the Helm chart using the following command:

`helm upgrade --install --repo https://trustification.io/trustify-helm-charts/ --devel -n $NAMESPACE trustify charts/trustify --values values-ocp-aws.yaml --set-string appDomain=$APP_DOMAIN`

The `--devel` flag is currently necessary as the Helm chart has a pre-release
version.

### Keycloak

Trustify requires an OIDC issuer. The recommended setup to use Keycloak as OIDC
issuer. For this, you will need to:

1. Install Keycloak
2. Create a new realm
3. Create the following roles for this realm
   - `chicken-user`
   - `chicken-manager`
   - `chicken-admin`
4. Make the `chicken-user` a default role
5. Create the following scopes for this realm
   - `read:document`
   - `create:document`
   - `delete:document`
6. Add the `create:document` and `delete:document` scope to the
   `chicken-manager` role
7. Create two clients
   - One public client
     - Set `standardFlowEnabled` to `true`
     - Set `fullScopedAllowed` to `true`
     - Set the following `defaultClientScopes`:
       - `read:document`
       - `create:document`
       - `delete:document`
   - One protected client
     - Set `publicClient` to `false`
     - Set `serviceAccountsEnabled` to `true`
     - Set `fullScopedAllowed` to `true`
     - Set the following `defaultClientScopes`:
       - `read:document`
       - `create:document`
     - Add role `chicken-manager` to the service account of this client
8. Increase the token timeout for both clients to at least 5 minutes
9. Create a user, acting as administrator

- Add the `chicken-manager` and `chicken-admin` role to this user
