# common-app

common-app is a unified chart template for deploy services with CI/CD.

## TL;DR

```console
$ git clone https://github.com/okassov/helm-chart-common-app.git
$ helm install my-release common-app/ --set image.repository=<Hostname of your container registry> --set image.tag=<tag of your container image>
```

## Prerequisites

- Kubernetes 1.12+
- Helm 3.1.0

## Installing the Chart

 To install the chart with the release name `my-release`:

```console
$ git clone https://github.com/okassov/helm-chart-common-app.git
$ helm install my-release common-app/ \
  --set image.repository=<Hostname of your container registry> \
  --set image.tag=<Tag of your container image>
```

These commands deploy service on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Parameters

### Global parameters

| Name                      | Description                                     | Value |
| ------------------------- | ----------------------------------------------- | ----- |
| `global.imagePullSecrets` | Global Docker registry secret names as an array | `[]`  |
| `global.storageClass`     | Global StorageClass for Persistent Volume(s)    | `nil` |


### Common parameters

| Name               | Description                                                                                               | Value |
| ------------------ | --------------------------------------------------------------------------------------------------------- | ----- |
| `kubeVersion`      | Force target Kubernetes version (using Helm capabilities if not set)                                      | `nil` |
| `nameOverride`     | String to partially override service.fullname template with a string (will prepend the release name)      | `nil` |
| `fullnameOverride` | String to fully override service.fullname template with a string                                          | `nil` |


### app parameters

| Name                                   | Description                                                                                                                                               | Value                    |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `image.registry`                       | App image registry                                                                                                                                        | `nil`                    |
| `image.repository`                     | App image repository                                                                                                                                      | `nil`                    |
| `image.tag`                            | App image tag (immutable tags are recommended)                                                                                                            | `nil`                    |
| `image.pullPolicy`                     | App image pull policy                                                                                                                                     | `Always`                 |
| `replicaCount`                         | Number of replicas of the app Pod                                                                                                                         | `1`                      |
| `updateStrategy.type`                  | Set up update strategy for app installation.                                                                                                              | `RollingUpdate`          |
| `hostAliases`                          | Add deployment host aliases                                                                                                                               | `[]`                     |
| `env`                                  | env vars to configure app                                                                                                                                 | `nil`                    |
| `envSecret`                            | Enable/Disabke ingest from secret containing extra env vars to configure app (in case of sensitive data)                                                  | `false`                  |
| `extraVolumes`                         | Array to add extra volumes. Requires setting `extraVolumeMounts`                                                                                          | `nil`                    |
| `extraVolumeMounts`                    | Array to add extra mounts. Normally used with `extraVolumes`                                                                                              | `nil`                    |
| `livenessProbe`                        | Liveness probe                                                                                                                                            | `nil`                    |
| `readinessProbe`                       | Readiness probe                                                                                                                                           | `nil`                    |
| `service.port`                         | Kubernetes Service port                                                                                                                                   | `8080`                   |
| `service.type`                         | Kubernetes Service type                                                                                                                                   | `ClusterIP`              |
| `service.nodePort`                     | Specify the nodePort value for the LoadBalancer and NodePort service types                                                                                | `nil`                    |
| `service.externalTrafficPolicy`        | Enable client source IP preservation                                                                                                                      | `Cluster`                |
| `service.loadBalancerIP`               | loadBalancerIP if app service type is `LoadBalancer`                                                                                                      | `nil`                    |
| `service.extraPorts`                   | Extra ports to expose in the service (normally used with the `sidecar` value)                                                                             | `nil`                    |
| `ingress.enabled`                      | Enable ingress controller resource                                                                                                                        | `false`                  |
| `ingress.certManager`                  | Add annotations for cert-manager                                                                                                                          | `false`                  |
| `ingress.pathType`                     | Ingress Path type                                                                                                                                         | `ImplementationSpecific` |
| `ingress.path`                         | The Path to app. You may need to set this to '/*' in order to use this with ALB ingress controllers.                                                      | `ImplementationSpecific` |
| `ingress.annotations`                  | Ingress annotations                                                                                                                                       | `{}`                     |
| `ingress.tls.secretName`               | TLS secret                                                                                                                                                | `false`                  |
| `ingress.tls.hosts`                    | TLS host                                                                                                                                                  | `false`                  |
| `ingress.extraHosts`                   | The list of additional hostnames to be covered with this ingress record.                                                                                  | `[]`                     |
| `ingress.extraPaths`                   | Additional arbitrary path/backend objects                                                                                                                 | `[]`                     |
| `ingress.extraTls`                     | The tls configuration for additional hostnames to be covered with this ingress record.                                                                    | `[]`                     |
| `containerPort`                        | Port to expose at container level                                                                                                                         | `8080`                   |
| `resources.limits`                     | The resources limits for the container                                                                                                                    | `{}`                     |
| `resources.requests`                   | The requested resources for the container                                                                                                                 | `{}`                     |
| `affinity`                             | Affinity for pod assignment                                                                                                                               | `{}`                     |
| `nodeSelector`                         | Node labels for pod assignment                                                                                                                            | `{}`                     |
| `tolerations`                          | Tolerations for pod assignment                                                                                                                            | `[]`                     |
| `sidecars`                             | Additional containers to run within the same pod                                                                                                          | `nil`                    |
| `initContainers`                       | Temporary init containers to run before main container up                                                                                                 | `nil`                    |


Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
$ helm install my-release -f values.yaml common-app/
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Configuration and installation details

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

### How to use environment variables

In case you want to add environment variables, you can use the `env` property.

```yaml
env:
  ELASTICSEARCH_VERSION: 6
  LOG_LEVEL: debug
```

### How to use secret environment variables

Alternatively, you can use a a Secret with the environment variables. To do so, use the `envSecret` values.

```yaml
envSecret: true
```

Then you enable this parameter (by default it's = false), your deployment will mount all data variables from secret in the same namespace. So before deploy application, you must to create application secret with variables. Secret name must be {{ service.fullname }}.

### Sidecars and Init Containers

If you have a need for additional containers to run within the same pod as app (e.g. an additional metrics or logging exporter), you can do so via the `sidecars` config parameter. Simply define your container according to the Kubernetes container spec.

```yaml
sidecars:
- name: your-image-name
  image: your-image
  imagePullPolicy: Always
  ports:
  - name: portname
    containerPort: 1234
```

Similarly, you can add extra init containers using the `initContainers` parameter.

```yaml
initContainers:
- name: your-image-name
  image: your-image
  imagePullPolicy: Always
  ports:
  - name: portname
    containerPort: 1234
```

### Adding extra volumes

The common-app chart supports mounting extra volumes (either PVCs, secrets or configmaps) by using the `extraVolumes` and `extraVolumeMounts` property. This can be combined with advanced operations like adding extra init containers and sidecars.

### Using with CI/CD

Each project must contain .helm folder with: values-{dev|staging|prod}.yaml

```console
$ export VALUES_FILE=.helm/values-${ENVIRONMENT}.yaml
$ cp ${VALUES_FILE} /tmp/input.yaml
$ envsubst < /tmp/input.yaml > ${VALUES_FILE}
$ helm upgrade --atomic --install ${CI_PROJECT_NAME} \
  --namespace test \
  --set image.repository="${REGISTRY_URL}/${CI_PROJECT_NAME}" \
  --set image.tag=${CI_COMMIT_SHORT_SHA} \
  --debug \
  --timeout 3m \
  --values ${VALUES_FILE}
```

Full CI/CD example:

```yaml
image: alpine/helm:latest

stages:
  - docker-build
  - deploy

variables:
  REGISTRY_URL: "dockerhub.io"

before_script:
  - export KUBECONFIG=/root/config
  - export HELM_PATH=.helm

docker-build-dev:
  stage: docker-build
  environment:
    name: dev
  script:
    - docker build -t ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_SHORT_SHA} .
    - docker push ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_SHORT_SHA}
  only:
    refs:
      - develop
      - /^feature\/.*$/
  
deploy-kube-dev:
  stage: deploy
  environment:
    name: dev
  variables:
    NAMESPACE: dev
  script:
    - echo ${KUBE_CONFIG_DEV} | base64 -d > /root/config
    - export VALUES_FILE=.helm/values-${ENVIRONMENT}.yaml
    - cp ${VALUES_FILE} /tmp/input.yaml
    - envsubst < /tmp/input.yaml > ${VALUES_FILE}
    - helm upgrade --atomic --install ${CI_PROJECT_NAME} --namespace=${NAMESPACE} --set image.repository=${REGISTRY_URL}/${CI_PROJECT_NAME} --set image.tag=${CI_COMMIT_SHORT_SHA} --debug --timeout 3m --values ${VALUES_FILE}
  only:
    refs:
      - develop
      - /^feature\/.*$/

docker-build-staging:
  stage: docker-build
  environment:
    name: staging
  script:
    - docker build -t ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_SHORT_SHA} .
    - docker push ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_SHORT_SHA}
  only:
    refs:
      - master
  
deploy-kube-staging:
  stage: deploy
  environment:
    name: staging
  variables:
    NAMESPACE: staging
  script:
    - echo ${KUBE_CONFIG_STAGING} | base64 -d > /root/config
    - export VALUES_FILE=.helm/values-${ENVIRONMENT}.yaml
    - cp ${VALUES_FILE} /tmp/input.yaml
    - envsubst < /tmp/input.yaml > ${VALUES_FILE}
    - helm upgrade --atomic --install ${CI_PROJECT_NAME} --namespace=${NAMESPACE} --set image.repository=${REGISTRY_URL}/${CI_PROJECT_NAME} --set image.tag=${CI_COMMIT_SHORT_SHA} --debug --timeout 3m --values ${VALUES_FILE}
  only:
    refs:
      - master

docker-build-prod:
  stage: docker-build
  environment:
    name: prod
  script:
    - docker build -t ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_TAG} .
    - docker push ${REGISTRY_URL}/${CI_PROJECT_NAME}:${CI_COMMIT_TAG}
  rules:
    - if: $CI_COMMIT_TAG
  
deploy-kube-prod:
  stage: deploy
  environment:
    name: prod
  variables:
    NAMESPACE: prod
  script:
    - echo ${KUBE_CONFIG_PROD} | base64 -d > /root/config
    - export VALUES_FILE=.helm/values-${ENVIRONMENT}.yaml
    - cp ${VALUES_FILE} /tmp/input.yaml
    - envsubst < /tmp/input.yaml > ${VALUES_FILE}
    - helm upgrade --atomic --install ${CI_PROJECT_NAME} --namespace=${NAMESPACE} --set image.repository=${REGISTRY_URL}/${CI_PROJECT_NAME} --set image.tag=${CI_COMMIT_TAG} --debug --timeout 3m --values ${VALUES_FILE}
  rules:
    - if: $CI_COMMIT_TAG
```