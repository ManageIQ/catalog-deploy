# Service Portal Service OpenShift Specs and Templates

The `bin` directory contains scripts which will:
 - Create and start builds of the service portal service images (`bin/build`)
 - Deploy the service portal images into the current namespaces (`bin/deploy`)
 - Remove the service portal service (`bin/teardown`)

The build and deploy scripts require a single parameter which should be a file containing template parameters for use with `oc process`

## Template Parameters
These are the parameters that can be used when building and deploying application

| Name                           | Description                                             | Default                          |
|--------------------------------|---------------------------------------------------------|----------------------------------|
| `APP_NAME`                     | Name of the application                                 | service-portal                   |
| `PATH_PREFIX`                  | API path to listen to                                   |                                  |
| `APPROVAL_HOST`                | Host to use for the approval URL                        |                                  |
| `APPROVAL_PORT`                | Port to use for the approval URL                        | 8080                             |
| `APPROVAL_SCHEME`              | Scheme to use for the approval URL                      | http                             |
| `IMAGE_NAMESPACE`              | Image namespace (used to build and deploy)              | buildfactory                     |
| `QUEUE_HOST`                   | Queue host/service name                                 |                                  |
| `TOPOLOGICAL_INVENTORY_HOST`   | Host to use for the topological inventory service URL   |                                  |
| `TOPOLOGICAL_INVENTORY_PORT`   | Port to use for the topological inventory service URL   | 8080                             |
| `TOPOLOGICAL_INVENTORY_SCHEME` | Scheme to use for the topological inventory service URL | http                             |
| `RBAC_HOST`                    | Host to use for the RBAC service URL                    |                                  |
| `RBAC_PORT`                    | Port to use for the RBAC service URL                    | 8080                             |
| `RBAC_SCHEME`                  | Scheme to use for the RBAC service URL                  | http                             |
| `DATABASE_PASSWORD`            | PostgreSQL database password                            | generated from "[a-zA-Z0-9]{8}"  |
| `SECRET_KEY`                   | Rails secret key                                        | generated from "[a-f0-9]{128}"   |
| `ENCRYPTION_KEY`               | Encryption key for passwords in the database            | generated from "[a-zA-Z0-9]{43}" |

---------------------------------------------------------------------------------------------------------

_**Notes on APP_NAME and PATH_PREFIX:**_

APP_NAME and PATH_PREFIX together will determine the base path that the API will listen on.
By default PATH_PREFIX is empty, if it is not specified, the application will listen on `/api/`
If PATH_PREFIX is specified, the application will listen on `/$PATH_PREFIX/$APP_NAME/`

## About Secrets

The deploy script is written so that it will not overwrite secrets.
This means that a secret will only be created or altered if it doesn't already exist or if that secret value is provided as a template parameter.

## Examples

These examples assume the approval service will be running in the same namespace and the topological inventory service will be running in a namespace called "topological-inventory" within the same cluster.

### Build and deploy in a namespace called service-portal with an internal queue
Parameters file (`params-file`) contains the following:

```plain
IMAGE_NAMESPACE=service-portal
APPROVAL_HOST=approval-api
TOPOLOGICAL_INVENTORY_HOST=topological-inventory-api.topological-inventory.svc
QUEUE_HOST=service-portal-kafka
RBAC_HOST=rbac.rbac.svc
```

1. Ensure you are working in the correct namespace

```bash
oc project service-portal
```

2. Create and run builds

```bash
bin/build params-file
```

3. Deploy the queue

```bash
oc apply -f queue/deployment_config.yaml
oc apply -f queue/service.yaml
```

4. Deploy all the other things

```bash
bin/deploy params-file
```

### Deploy from buildfactory with an external queue service and a path prefix
Parameters file (`params-file`) contains the following:

```plain
APPROVAL_HOST=approval-api
TOPOLOGICAL_INVENTORY_HOST=topological-inventory-api.topological-inventory.svc.cluster.local
QUEUE_HOST=platform-mq-ci-kafka-bootstrap.platform-mq-ci.svc
PATH_PREFIX=r/insights/platform
RBAC_HOST=rbac.rbac-ci.svc.cluster.local
```

1. Ensure you are working in the correct namespace

```bash
oc project service-portal
```

2. Deploy the things

```bash
bin/deploy params-file
```
