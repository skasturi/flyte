# flyte

![Version: v0.1.10](https://img.shields.io/badge/Version-v0.1.10-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

A Helm chart for Flyte

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | contour | 4.1.2 |
| https://googlecloudplatform.github.io/spark-on-k8s-operator | sparkoperator(spark-operator) | 1.0.6 |
| https://kubernetes.github.io/dashboard/ | kubernetes-dashboard | 4.0.2 |

### Flyte INSTALLATION:
- [Install helm 3](https://helm.sh/docs/intro/install/)
- Fetch chart dependencies ``
- Install Flyte:

```bash
helm repo add flyte https://flyteorg.github.io/flyte
helm install -n flyte -f values-eks.yaml --create-namespace flyte flyte/flyte-core
```

Customize your installation by changing settings in `values-eks.yaml`.
You can use the helm diff plugin to review any value changes you've made to your values:

```bash
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade -f values-eks.yaml flyte flyte/flyte-core
```

Then apply your changes:
```bash
helm upgrade -f values-eks.yaml flyte flyte/flyte-core
```

Install ingress controller (By default Flyte helm chart have contour ingress resource)
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install gateway bitnami/contour -n flyte
```

#### Alternative: Generate raw kubernetes yaml with helm template
- `helm template --name-template=flyte-eks . -n flyte -f values-eks.yaml > flyte_generated_eks.yaml`
- Deploy the manifest `kubectl apply -f flyte_generated_eks.yaml`

- When all pods are running - run end2end tests: `kubectl apply -f ../end2end/tests/endtoend.yaml`
- Get flyte host `minikube service contour -n heptio-contour --url`. And then visit `http://<HOST>/console`

### CONFIGURATION NOTES:
- The docker images, their tags and other default parameters are configured in `values.yaml` file.
- Each Flyte installation type should have separate `values-*.yaml` file: for sandbox, EKS and etc. The configuration in `values.yaml` and the choosen config `values-*.yaml` are merged when generating the deployment manifest.
- The configuration in `values-sandbox.yaml` is ready for installation in minikube. But `values-eks.yaml` should be edited before installation: s3 bucket, RDS hosts, iam roles, secrets and etc need to be modified.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cluster_resource_manager | object | `{"config":{"cluster_resources":{"customData":[{"production":[{"projectQuotaCpu":{"value":"5"}},{"projectQuotaMemory":{"value":"4000Mi"}},{"defaultIamRole":{"value":""}}]},{"staging":[{"projectQuotaCpu":{"value":"2"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]},{"development":[{"projectQuotaCpu":{"value":"4"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]}],"refresh":"5m","refreshInterval":"5m","templatePath":"/etc/flyte/clusterresource/templates"}},"enabled":true,"templates":[{"key":"aa_namespace","value":"apiVersion: v1\nkind: Namespace\nmetadata:\n  name: {{ namespace }}\nspec:\n  finalizers:\n  - kubernetes\n"},{"key":"ab_project_resource_quota","value":"apiVersion: v1\nkind: ResourceQuota\nmetadata:\n  name: project-quota\n  namespace: {{ namespace }}\nspec:\n  hard:\n    limits.cpu: {{ projectQuotaCpu }}\n    limits.memory: {{ projectQuotaMemory }}\n"}]}` | Configuration for the Cluster resource manager component. This is an optional component, that enables automatic cluster configuration. This is useful to set default quotas, manage namespaces etc that map to a project/domain |
| cluster_resource_manager.config | object | `{"cluster_resources":{"customData":[{"production":[{"projectQuotaCpu":{"value":"5"}},{"projectQuotaMemory":{"value":"4000Mi"}},{"defaultIamRole":{"value":""}}]},{"staging":[{"projectQuotaCpu":{"value":"2"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]},{"development":[{"projectQuotaCpu":{"value":"4"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]}],"refresh":"5m","refreshInterval":"5m","templatePath":"/etc/flyte/clusterresource/templates"}}` | Configmap for ClusterResource parameters |
| cluster_resource_manager.config.cluster_resources | object | `{"customData":[{"production":[{"projectQuotaCpu":{"value":"5"}},{"projectQuotaMemory":{"value":"4000Mi"}},{"defaultIamRole":{"value":""}}]},{"staging":[{"projectQuotaCpu":{"value":"2"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]},{"development":[{"projectQuotaCpu":{"value":"4"}},{"projectQuotaMemory":{"value":"3000Mi"}},{"defaultIamRole":{"value":""}}]}],"refresh":"5m","refreshInterval":"5m","templatePath":"/etc/flyte/clusterresource/templates"}` | ClusterResource parameters Refer to the [structure](https://pkg.go.dev/github.com/lyft/flyteadmin@v0.3.37/pkg/runtime/interfaces#ClusterResourceConfig) to customize. |
| cluster_resource_manager.enabled | bool | `true` | Enables the Cluster resource manager component |
| cluster_resource_manager.templates | list | `[{"key":"aa_namespace","value":"apiVersion: v1\nkind: Namespace\nmetadata:\n  name: {{ namespace }}\nspec:\n  finalizers:\n  - kubernetes\n"},{"key":"ab_project_resource_quota","value":"apiVersion: v1\nkind: ResourceQuota\nmetadata:\n  name: project-quota\n  namespace: {{ namespace }}\nspec:\n  hard:\n    limits.cpu: {{ projectQuotaCpu }}\n    limits.memory: {{ projectQuotaMemory }}\n"}]` | Resource templates that should be applied |
| cluster_resource_manager.templates[0] | object | `{"key":"aa_namespace","value":"apiVersion: v1\nkind: Namespace\nmetadata:\n  name: {{ namespace }}\nspec:\n  finalizers:\n  - kubernetes\n"}` | Template for namespaces resources |
| common | object | `{"databaseSecret":{"name":"","secretManifest":{}},"flyteNamespaceTemplate":{"enabled":false},"ingress":{"albSSLRedirect":false,"annotations":{},"enabled":true,"separateGrpcIngress":false,"separateGrpcIngressAnnotations":{"nginx.ingress.kubernetes.io/backend-protocol":"GRPC"},"tls":{"enabled":false},"webpackHMR":false}}` | ---------------------------------------------- COMMON SETTINGS |
| common.databaseSecret.name | string | `""` | Specify name of K8s Secret which contains Database password. Leave it empty if you don't need this Secret |
| common.databaseSecret.secretManifest | object | `{}` | Specify your Secret (with sensitive data) or pseudo-manifest (without sensitive data). See https://github.com/godaddy/kubernetes-external-secrets |
| common.flyteNamespaceTemplate.enabled | bool | `false` | - Enable or disable creating Flyte namespace in template. Enable when using helm as template-engine only. Disable when using `helm install ...`. |
| common.ingress.albSSLRedirect | bool | `false` | - albSSLRedirect adds a special route for ssl redirect. Only useful in combination with the AWS LoadBalancer Controller. |
| common.ingress.annotations | object | `{}` | - Ingress annotations applied to both HTTP and GRPC ingresses. |
| common.ingress.enabled | bool | `true` | - Enable or disable creating Ingress for Flyte. Relevant to disable when using e.g. Istio as ingress controller. |
| common.ingress.separateGrpcIngress | bool | `false` | - separateGrpcIngress puts GRPC routes into a separate ingress if true. Required for certain ingress controllers like nginx. |
| common.ingress.separateGrpcIngressAnnotations | object | `{"nginx.ingress.kubernetes.io/backend-protocol":"GRPC"}` | - Extra Ingress annotations applied only to the GRPC ingress. Only makes sense if `separateGrpcIngress` is enabled. |
| common.ingress.tls | object | `{"enabled":false}` | - Ingress hostname host: |
| common.ingress.webpackHMR | bool | `false` | - Enable or disable HMR route to flyteconsole. This is useful only for frontend development. |
| configmap | object | `{"admin":{"admin":{"clientId":"flytepropeller","clientSecretLocation":"/etc/secrets/client_secret","endpoint":"flyteadmin:81","insecure":true},"event":{"capacity":1000,"rate":500,"type":"admin"}},"adminServer":{"auth":{"appAuth":{"thirdPartyConfig":{"flyteClient":{"clientId":"flytectl","redirectUri":"http://localhost:53593/callback","scopes":["offline","all"]}}},"authorizedUris":["https://localhost:30081","http://flyteadmin:80","http://flyteadmin.flyte.svc.cluster.local:80"],"userAuth":{"openId":{"baseUrl":"https://accounts.google.com","clientId":"657465813211-6eog7ek7li5k7i7fvgv2921075063hpe.apps.googleusercontent.com","scopes":["profile","openid"]}}},"flyteadmin":{"eventVersion":1,"metadataStoragePrefix":["metadata","admin"],"metricsScope":"flyte:","profilerPort":10254,"roleNameKey":"iam.amazonaws.com/role","testing":{"host":"http://flyteadmin"}},"server":{"grpcPort":8089,"httpPort":8088,"security":{"allowCors":true,"allowedHeaders":["Content-Type","flyte-authorization"],"allowedOrigins":["*"],"secure":false,"useAuth":false}}},"catalog":{"catalog-cache":{"endpoint":"datacatalog:89","insecure":true,"type":"datacatalog"}},"console":{"BASE_URL":"/console","CONFIG_DIR":"/etc/flyte/config","DISABLE_AUTH":"1"},"copilot":{"plugins":{"k8s":{"co-pilot":{"image":"cr.flyte.org/lyft/flyteplugins/flytecopilot:dc4bdbd61cac88a39a5ff43e40f026bdbc2c78a2","name":"flyte-copilot-","start-timeout":"30s"}}}},"core":{"propeller":{"downstream-eval-duration":"30s","enable-admin-launcher":true,"leader-election":{"enabled":true,"lease-duration":"15s","lock-config-map":{"name":"propeller-leader","namespace":"flyte"},"renew-deadline":"10s","retry-period":"2s"},"limit-namespace":"all","max-workflow-retries":30,"metadata-prefix":"metadata/propeller","metrics-prefix":"flyte","prof-port":10254,"queue":{"batch-size":-1,"batching-interval":"2s","queue":{"base-delay":"5s","capacity":1000,"max-delay":"120s","rate":100,"type":"maxof"},"sub-queue":{"capacity":100,"rate":10,"type":"bucket"},"type":"batch"},"rawoutput-prefix":"s3://my-s3-bucket/","workers":4,"workflow-reeval-duration":"30s"},"webhook":{"certDir":"/etc/webhook/certs","serviceName":"flyte-pod-webhook"}},"datacatalogServer":{"application":{"grpcPort":8089,"grpcServerReflection":true,"httpPort":8080},"datacatalog":{"metrics-scope":"datacatalog","profiler-port":10254,"storage-prefix":"metadata/datacatalog"}},"domain":{"domains":[{"id":"development","name":"development"},{"id":"staging","name":"staging"},{"id":"production","name":"production"}]},"enabled_plugins":{"tasks":{"task-plugins":{"default-for-task-types":{"container":"container","container_array":"k8s-array","sidecar":"sidecar"},"enabled-plugins":["container","sidecar","k8s-array"]}}},"k8s":{"plugins":{"k8s":{"default-cpus":"100m","default-env-vars":[],"default-memory":"100Mi"}}},"logger":{"logger":{"level":4,"show-source":true}},"remoteData":{"remoteData":{"region":"us-east-1","scheme":"local","signedUrls":{"durationMinutes":3}}},"resource_manager":{"propeller":{"resourcemanager":{"redis":{"hostKey":"mypassword","hostPath":"redis-resource-manager:6379"},"resourceMaxQuota":10000,"type":"redis"}}},"task_logs":{"plugins":{"logs":{"cloudwatch-enabled":false,"kubernetes-enabled":false}}},"task_resource_defaults":{"task_resources":{"defaults":{"cpu":"100m","memory":"100Mi","storage":"5Mi"},"limits":{"cpu":2,"gpu":1,"memory":"1Gi","storage":"20Mi"}}}}` | ------------------------------------------------------------------ Specializing your deployment using configuration ------------------------------------------------------------------- CONFIGMAPS SETTINGS |
| configmap.admin | object | `{"admin":{"clientId":"flytepropeller","clientSecretLocation":"/etc/secrets/client_secret","endpoint":"flyteadmin:81","insecure":true},"event":{"capacity":1000,"rate":500,"type":"admin"}}` | Admin Client configuration [structure](https://pkg.go.dev/github.com/flyteorg/flytepropeller/pkg/controller/nodes/subworkflow/launchplan#AdminConfig) |
| configmap.adminServer | object | `{"auth":{"appAuth":{"thirdPartyConfig":{"flyteClient":{"clientId":"flytectl","redirectUri":"http://localhost:53593/callback","scopes":["offline","all"]}}},"authorizedUris":["https://localhost:30081","http://flyteadmin:80","http://flyteadmin.flyte.svc.cluster.local:80"],"userAuth":{"openId":{"baseUrl":"https://accounts.google.com","clientId":"657465813211-6eog7ek7li5k7i7fvgv2921075063hpe.apps.googleusercontent.com","scopes":["profile","openid"]}}},"flyteadmin":{"eventVersion":1,"metadataStoragePrefix":["metadata","admin"],"metricsScope":"flyte:","profilerPort":10254,"roleNameKey":"iam.amazonaws.com/role","testing":{"host":"http://flyteadmin"}},"server":{"grpcPort":8089,"httpPort":8088,"security":{"allowCors":true,"allowedHeaders":["Content-Type","flyte-authorization"],"allowedOrigins":["*"],"secure":false,"useAuth":false}}}` | FlyteAdmin server configuration |
| configmap.adminServer.auth | object | `{"appAuth":{"thirdPartyConfig":{"flyteClient":{"clientId":"flytectl","redirectUri":"http://localhost:53593/callback","scopes":["offline","all"]}}},"authorizedUris":["https://localhost:30081","http://flyteadmin:80","http://flyteadmin.flyte.svc.cluster.local:80"],"userAuth":{"openId":{"baseUrl":"https://accounts.google.com","clientId":"657465813211-6eog7ek7li5k7i7fvgv2921075063hpe.apps.googleusercontent.com","scopes":["profile","openid"]}}}` | Authentication configuration |
| configmap.adminServer.server.security.secure | bool | `false` | Controls whether to serve requests over SSL/TLS. |
| configmap.adminServer.server.security.useAuth | bool | `false` | Controls whether to enforce authentication. Follow the guide in https://docs.flyte.org/ on how to setup authentication. |
| configmap.catalog | object | `{"catalog-cache":{"endpoint":"datacatalog:89","insecure":true,"type":"datacatalog"}}` | Catalog Client configuration [structure](https://pkg.go.dev/github.com/flyteorg/flytepropeller/pkg/controller/nodes/task/catalog#Config) Additional advanced Catalog configuration [here](https://pkg.go.dev/github.com/lyft/flyteplugins/go/tasks/pluginmachinery/catalog#Config) |
| configmap.console | object | `{"BASE_URL":"/console","CONFIG_DIR":"/etc/flyte/config","DISABLE_AUTH":"1"}` | Configuration for Flyte console UI |
| configmap.copilot | object | `{"plugins":{"k8s":{"co-pilot":{"image":"cr.flyte.org/lyft/flyteplugins/flytecopilot:dc4bdbd61cac88a39a5ff43e40f026bdbc2c78a2","name":"flyte-copilot-","start-timeout":"30s"}}}}` | Copilot configuration |
| configmap.copilot.plugins.k8s.co-pilot | object | `{"image":"cr.flyte.org/lyft/flyteplugins/flytecopilot:dc4bdbd61cac88a39a5ff43e40f026bdbc2c78a2","name":"flyte-copilot-","start-timeout":"30s"}` | Structure documented [here](https://pkg.go.dev/github.com/lyft/flyteplugins@v0.5.28/go/tasks/pluginmachinery/flytek8s/config#FlyteCoPilotConfig) |
| configmap.core | object | `{"propeller":{"downstream-eval-duration":"30s","enable-admin-launcher":true,"leader-election":{"enabled":true,"lease-duration":"15s","lock-config-map":{"name":"propeller-leader","namespace":"flyte"},"renew-deadline":"10s","retry-period":"2s"},"limit-namespace":"all","max-workflow-retries":30,"metadata-prefix":"metadata/propeller","metrics-prefix":"flyte","prof-port":10254,"queue":{"batch-size":-1,"batching-interval":"2s","queue":{"base-delay":"5s","capacity":1000,"max-delay":"120s","rate":100,"type":"maxof"},"sub-queue":{"capacity":100,"rate":10,"type":"bucket"},"type":"batch"},"rawoutput-prefix":"s3://my-s3-bucket/","workers":4,"workflow-reeval-duration":"30s"},"webhook":{"certDir":"/etc/webhook/certs","serviceName":"flyte-pod-webhook"}}` | Core propeller configuration |
| configmap.core.propeller | object | `{"downstream-eval-duration":"30s","enable-admin-launcher":true,"leader-election":{"enabled":true,"lease-duration":"15s","lock-config-map":{"name":"propeller-leader","namespace":"flyte"},"renew-deadline":"10s","retry-period":"2s"},"limit-namespace":"all","max-workflow-retries":30,"metadata-prefix":"metadata/propeller","metrics-prefix":"flyte","prof-port":10254,"queue":{"batch-size":-1,"batching-interval":"2s","queue":{"base-delay":"5s","capacity":1000,"max-delay":"120s","rate":100,"type":"maxof"},"sub-queue":{"capacity":100,"rate":10,"type":"bucket"},"type":"batch"},"rawoutput-prefix":"s3://my-s3-bucket/","workers":4,"workflow-reeval-duration":"30s"}` | follows the structure specified [here](https://pkg.go.dev/github.com/flyteorg/flytepropeller/pkg/controller/config). |
| configmap.datacatalogServer | object | `{"application":{"grpcPort":8089,"grpcServerReflection":true,"httpPort":8080},"datacatalog":{"metrics-scope":"datacatalog","profiler-port":10254,"storage-prefix":"metadata/datacatalog"}}` | Datacatalog server config |
| configmap.domain | object | `{"domains":[{"id":"development","name":"development"},{"id":"staging","name":"staging"},{"id":"production","name":"production"}]}` | Domains configuration for Flyte projects. This enables the specified number of domains across all projects in Flyte. |
| configmap.enabled_plugins.tasks | object | `{"task-plugins":{"default-for-task-types":{"container":"container","container_array":"k8s-array","sidecar":"sidecar"},"enabled-plugins":["container","sidecar","k8s-array"]}}` | Tasks specific configuration [structure](https://pkg.go.dev/github.com/flyteorg/flytepropeller/pkg/controller/nodes/task/config#GetConfig) |
| configmap.enabled_plugins.tasks.task-plugins | object | `{"default-for-task-types":{"container":"container","container_array":"k8s-array","sidecar":"sidecar"},"enabled-plugins":["container","sidecar","k8s-array"]}` | Plugins configuration, [structure](https://pkg.go.dev/github.com/flyteorg/flytepropeller/pkg/controller/nodes/task/config#TaskPluginConfig) |
| configmap.enabled_plugins.tasks.task-plugins.enabled-plugins | list | `["container","sidecar","k8s-array"]` | [Enabled Plugins](https://pkg.go.dev/github.com/lyft/flyteplugins/go/tasks/config#Config). Enable sagemaker*, athena if you install the backend plugins |
| configmap.k8s | object | `{"plugins":{"k8s":{"default-cpus":"100m","default-env-vars":[],"default-memory":"100Mi"}}}` | Kubernetes specific Flyte configuration |
| configmap.k8s.plugins.k8s | object | `{"default-cpus":"100m","default-env-vars":[],"default-memory":"100Mi"}` | Configuration section for all K8s specific plugins [Configuration structure](https://pkg.go.dev/github.com/lyft/flyteplugins/go/tasks/pluginmachinery/flytek8s/config) |
| configmap.logger | object | `{"logger":{"level":4,"show-source":true}}` | Logger configuration |
| configmap.resource_manager | object | `{"propeller":{"resourcemanager":{"redis":{"hostKey":"mypassword","hostPath":"redis-resource-manager:6379"},"resourceMaxQuota":10000,"type":"redis"}}}` | Resource manager configuration |
| configmap.resource_manager.propeller | object | `{"resourcemanager":{"redis":{"hostKey":"mypassword","hostPath":"redis-resource-manager:6379"},"resourceMaxQuota":10000,"type":"redis"}}` | resource manager configuration |
| configmap.task_logs | object | `{"plugins":{"logs":{"cloudwatch-enabled":false,"kubernetes-enabled":false}}}` | Section that configures how the Task logs are displayed on the UI. This has to be changed based on your actual logging provider. Refer to [structure](https://pkg.go.dev/github.com/lyft/flyteplugins/go/tasks/logs#LogConfig) to understand how to configure various logging engines |
| configmap.task_logs.plugins.logs.cloudwatch-enabled | bool | `false` | One option is to enable cloudwatch logging for EKS, update the region and log group accordingly |
| configmap.task_resource_defaults | object | `{"task_resources":{"defaults":{"cpu":"100m","memory":"100Mi","storage":"5Mi"},"limits":{"cpu":2,"gpu":1,"memory":"1Gi","storage":"20Mi"}}}` | Task default resources configuration Refer to the full [structure](https://pkg.go.dev/github.com/lyft/flyteadmin@v0.3.37/pkg/runtime/interfaces#TaskResourceConfiguration). |
| configmap.task_resource_defaults.task_resources | object | `{"defaults":{"cpu":"100m","memory":"100Mi","storage":"5Mi"},"limits":{"cpu":2,"gpu":1,"memory":"1Gi","storage":"20Mi"}}` | Task default resources parameters |
| contour | object | `{"affinity":{},"contour":{"resources":{"limits":{"cpu":"100m","memory":"100Mi"},"requests":{"cpu":"10m","memory":"50Mi"}}},"enabled":true,"envoy":{"resources":{"limits":{"cpu":"100m","memory":"100Mi"},"requests":{"cpu":"10m","memory":"50Mi"}}},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"serviceAccountAnnotations":{},"tolerations":[]}` |  CONTOUR SETTINGS |
| contour.affinity | object | `{}` | affinity for Contour deployment |
| contour.contour.resources | object | `{"limits":{"cpu":"100m","memory":"100Mi"},"requests":{"cpu":"10m","memory":"50Mi"}}` | Default resources requests and limits for Contour |
| contour.contour.resources.limits | object | `{"cpu":"100m","memory":"100Mi"}` | Limits are the maximum set of resources needed for this pod |
| contour.contour.resources.requests | object | `{"cpu":"10m","memory":"50Mi"}` | Requests are the minimum set of resources needed for this pod |
| contour.enabled | bool | `true` | - enable or disable Contour deployment installation |
| contour.envoy.resources | object | `{"limits":{"cpu":"100m","memory":"100Mi"},"requests":{"cpu":"10m","memory":"50Mi"}}` | Default resources requests and limits for Envoy |
| contour.envoy.resources.limits | object | `{"cpu":"100m","memory":"100Mi"}` | Limits are the maximum set of resources needed for this pod |
| contour.envoy.resources.requests | object | `{"cpu":"10m","memory":"50Mi"}` | Requests are the minimum set of resources needed for this pod |
| contour.nodeSelector | object | `{}` | nodeSelector for Contour deployment |
| contour.podAnnotations | object | `{}` | Annotations for Contour pods |
| contour.replicaCount | int | `1` | Replicas count for Contour deployment |
| contour.serviceAccountAnnotations | object | `{}` | Annotations for ServiceAccount attached to Contour pods |
| contour.tolerations | list | `[]` | tolerations for Contour deployment |
| datacatalog | object | `{"affinity":{},"configPath":"/etc/datacatalog/config/*.yaml","image":{"pullPolicy":"IfNotPresent","repository":"cr.flyte.org/flyteorg/datacatalog","tag":"v0.3.9"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"500m","ephemeral-storage":"100Mi","memory":"500Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}},"service":{"annotations":{"projectcontour.io/upstream-protocol.h2c":"grpc"},"type":"NodePort"},"serviceAccount":{"annotations":{},"create":true,"imagePullSecrets":{}},"tolerations":[]}` |  DATACATALOG SETTINGS |
| datacatalog.affinity | object | `{}` | affinity for Datacatalog deployment |
| datacatalog.configPath | string | `"/etc/datacatalog/config/*.yaml"` | Default regex string for searching configuration files |
| datacatalog.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| datacatalog.image.repository | string | `"cr.flyte.org/flyteorg/datacatalog"` | Docker image for Datacatalog deployment |
| datacatalog.image.tag | string | `"v0.3.9"` | Docker image tag |
| datacatalog.nodeSelector | object | `{}` | nodeSelector for Datacatalog deployment |
| datacatalog.podAnnotations | object | `{}` | Annotations for Datacatalog pods |
| datacatalog.replicaCount | int | `1` | Replicas count for Datacatalog deployment |
| datacatalog.resources | object | `{"limits":{"cpu":"500m","ephemeral-storage":"100Mi","memory":"500Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}}` | Default resources requests and limits for Datacatalog deployment |
| datacatalog.service | object | `{"annotations":{"projectcontour.io/upstream-protocol.h2c":"grpc"},"type":"NodePort"}` | Service settings for Datacatalog |
| datacatalog.serviceAccount | object | `{"annotations":{},"create":true,"imagePullSecrets":{}}` | Configuration for service accounts for Datacatalog |
| datacatalog.serviceAccount.annotations | object | `{}` | Annotations for ServiceAccount attached to Datacatalog pods |
| datacatalog.serviceAccount.create | bool | `true` | Should a service account be created for Datacatalog |
| datacatalog.serviceAccount.imagePullSecrets | object | `{}` | ImapgePullSecrets to automatically assign to the service account |
| datacatalog.tolerations | list | `[]` | tolerations for Datacatalog deployment |
| db.admin.database.dbname | string | `"flyteadmin"` |  |
| db.admin.database.host | string | `"postgres"` |  |
| db.admin.database.port | int | `5432` |  |
| db.admin.database.username | string | `"postgres"` |  |
| db.datacatalog.database.dbname | string | `"datacatalog"` |  |
| db.datacatalog.database.host | string | `"postgres"` |  |
| db.datacatalog.database.port | int | `5432` |  |
| db.datacatalog.database.username | string | `"postgres"` |  |
| flyteadmin.additionalVolumeMounts | list | `[]` |  |
| flyteadmin.additionalVolumes | list | `[]` |  |
| flyteadmin.affinity | object | `{}` | affinity for Flyteadmin deployment |
| flyteadmin.configPath | string | `"/etc/flyte/config/*.yaml"` | Default regex string for searching configuration files |
| flyteadmin.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| flyteadmin.image.repository | string | `"cr.flyte.org/flyteorg/flyteadmin"` | Docker image for Flyteadmin deployment |
| flyteadmin.image.tag | string | `"v0.6.28"` | Docker image tag |
| flyteadmin.initialProjects | list | `["flytesnacks","flytetester","flyteexamples"]` | Initial projects to create |
| flyteadmin.nodeSelector | object | `{}` | nodeSelector for Flyteadmin deployment |
| flyteadmin.podAnnotations | object | `{}` | Annotations for Flyteadmin pods |
| flyteadmin.replicaCount | int | `1` | Replicas count for Flyteadmin deployment |
| flyteadmin.resources | object | `{"limits":{"cpu":"250m","ephemeral-storage":"100Mi","memory":"500Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}}` | Default resources requests and limits for Flyteadmin deployment |
| flyteadmin.secrets | object | `{}` |  |
| flyteadmin.service | object | `{"annotations":{"projectcontour.io/upstream-protocol.h2c":"grpc"},"loadBalancerSourceRanges":[],"type":"ClusterIP"}` | Service settings for Flyteadmin |
| flyteadmin.serviceAccount | object | `{"annotations":{},"create":true,"imagePullSecrets":{}}` | Configuration for service accounts for FlyteAdmin |
| flyteadmin.serviceAccount.annotations | object | `{}` | Annotations for ServiceAccount attached to Flyteadmin pods |
| flyteadmin.serviceAccount.create | bool | `true` | Should a service account be created for flyteadmin |
| flyteadmin.serviceAccount.imagePullSecrets | object | `{}` | ImapgePullSecrets to automatically assign to the service account |
| flyteadmin.tolerations | list | `[]` | tolerations for Flyteadmin deployment |
| flyteconsole | object | `{"affinity":{},"image":{"pullPolicy":"IfNotPresent","repository":"cr.flyte.org/flyteorg/flyteconsole","tag":"v0.25.1"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"500m","memory":"250Mi"},"requests":{"cpu":"10m","memory":"50Mi"}},"service":{"annotations":{},"type":"ClusterIP"},"tolerations":[]}` |  FLYTECONSOLE SETTINGS |
| flyteconsole.affinity | object | `{}` | affinity for Flyteconsole deployment |
| flyteconsole.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| flyteconsole.image.repository | string | `"cr.flyte.org/flyteorg/flyteconsole"` | Docker image for Flyteconsole deployment |
| flyteconsole.image.tag | string | `"v0.25.1"` | Docker image tag |
| flyteconsole.nodeSelector | object | `{}` | nodeSelector for Flyteconsole deployment |
| flyteconsole.podAnnotations | object | `{}` | Annotations for Flyteconsole pods |
| flyteconsole.replicaCount | int | `1` | Replicas count for Flyteconsole deployment |
| flyteconsole.resources | object | `{"limits":{"cpu":"500m","memory":"250Mi"},"requests":{"cpu":"10m","memory":"50Mi"}}` | Default resources requests and limits for Flyteconsole deployment |
| flyteconsole.service | object | `{"annotations":{},"type":"ClusterIP"}` | Service settings for Flyteconsole |
| flyteconsole.tolerations | list | `[]` | tolerations for Flyteconsole deployment |
| flytepropeller | object | `{"affinity":{},"cacheSizeMbs":0,"configPath":"/etc/flyte/config/*.yaml","image":{"pullPolicy":"IfNotPresent","repository":"cr.flyte.org/flyteorg/flytepropeller","tag":"v0.14.3"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"200m","ephemeral-storage":"100Mi","memory":"200Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}},"serviceAccount":{"annotations":{},"create":true,"imagePullSecrets":{}},"tolerations":[]}` |  FLYTEPROPELLER SETTINGS |
| flytepropeller.affinity | object | `{}` | affinity for Flytepropeller deployment |
| flytepropeller.configPath | string | `"/etc/flyte/config/*.yaml"` | Default regex string for searching configuration files |
| flytepropeller.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| flytepropeller.image.repository | string | `"cr.flyte.org/flyteorg/flytepropeller"` | Docker image for Flytepropeller deployment |
| flytepropeller.image.tag | string | `"v0.14.3"` | Docker image tag |
| flytepropeller.nodeSelector | object | `{}` | nodeSelector for Flytepropeller deployment |
| flytepropeller.podAnnotations | object | `{}` | Annotations for Flytepropeller pods |
| flytepropeller.replicaCount | int | `1` | Replicas count for Flytepropeller deployment |
| flytepropeller.resources | object | `{"limits":{"cpu":"200m","ephemeral-storage":"100Mi","memory":"200Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}}` | Default resources requests and limits for Flytepropeller deployment |
| flytepropeller.serviceAccount | object | `{"annotations":{},"create":true,"imagePullSecrets":{}}` | Configuration for service accounts for FlytePropeller |
| flytepropeller.serviceAccount.annotations | object | `{}` | Annotations for ServiceAccount attached to FlytePropeller pods |
| flytepropeller.serviceAccount.create | bool | `true` | Should a service account be created for FlytePropeller |
| flytepropeller.serviceAccount.imagePullSecrets | object | `{}` | ImapgePullSecrets to automatically assign to the service account |
| flytepropeller.tolerations | list | `[]` | tolerations for Flytepropeller deployment |
| flytescheduler | object | `{"affinity":{},"configPath":"/etc/flyte/config/*.yaml","image":{"pullPolicy":"IfNotPresent","repository":"cr.flyte.org/flyteorg/flytescheduler","tag":"v0.6.28"},"nodeSelector":{},"podAnnotations":{},"resources":{"limits":{"cpu":"250m","ephemeral-storage":"100Mi","memory":"500Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}},"secrets":{},"serviceAccount":{"annotations":{},"create":true,"imagePullSecrets":{}},"tolerations":[]}` |  FLYTESCHEDULER SETTINGS |
| flytescheduler.affinity | object | `{}` | affinity for Flytescheduler deployment |
| flytescheduler.configPath | string | `"/etc/flyte/config/*.yaml"` | Default regex string for searching configuration files |
| flytescheduler.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| flytescheduler.image.repository | string | `"cr.flyte.org/flyteorg/flytescheduler"` | Docker image for Flytescheduler deployment |
| flytescheduler.image.tag | string | `"v0.6.28"` | Docker image tag |
| flytescheduler.nodeSelector | object | `{}` | nodeSelector for Flytescheduler deployment |
| flytescheduler.podAnnotations | object | `{}` | Annotations for Flytescheduler pods |
| flytescheduler.resources | object | `{"limits":{"cpu":"250m","ephemeral-storage":"100Mi","memory":"500Mi"},"requests":{"cpu":"10m","ephemeral-storage":"50Mi","memory":"50Mi"}}` | Default resources requests and limits for Flytescheduler deployment |
| flytescheduler.serviceAccount | object | `{"annotations":{},"create":true,"imagePullSecrets":{}}` | Configuration for service accounts for Flytescheduler |
| flytescheduler.serviceAccount.annotations | object | `{}` | Annotations for ServiceAccount attached to Flytescheduler pods |
| flytescheduler.serviceAccount.create | bool | `true` | Should a service account be created for Flytescheduler |
| flytescheduler.serviceAccount.imagePullSecrets | object | `{}` | ImapgePullSecrets to automatically assign to the service account |
| flytescheduler.tolerations | list | `[]` | tolerations for Flytescheduler deployment |
| kubernetes-dashboard | object | `{"enabled":false}` |  KUBERNETES DASHBOARD |
| minio | object | `{"affinity":{},"enabled":true,"image":{"pullPolicy":"IfNotPresent","repository":"minio/minio","tag":"RELEASE.2020-12-16T05-05-17Z"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"200m","memory":"512Mi"},"requests":{"cpu":"10m","memory":"128Mi"}},"service":{"annotations":{},"type":"ClusterIP"},"tolerations":[]}` |  MINIO SETTINGS |
| minio.affinity | object | `{}` | affinity for Minio deployment |
| minio.enabled | bool | `true` | - enable or disable Minio deployment installation |
| minio.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| minio.image.repository | string | `"minio/minio"` | Docker image for Minio deployment |
| minio.image.tag | string | `"RELEASE.2020-12-16T05-05-17Z"` | Docker image tag |
| minio.nodeSelector | object | `{}` | nodeSelector for Minio deployment |
| minio.podAnnotations | object | `{}` | Annotations for Minio pods |
| minio.replicaCount | int | `1` | Replicas count for Minio deployment |
| minio.resources | object | `{"limits":{"cpu":"200m","memory":"512Mi"},"requests":{"cpu":"10m","memory":"128Mi"}}` | Default resources requests and limits for Minio deployment |
| minio.resources.limits | object | `{"cpu":"200m","memory":"512Mi"}` | Limits are the maximum set of resources needed for this pod |
| minio.resources.requests | object | `{"cpu":"10m","memory":"128Mi"}` | Requests are the minimum set of resources needed for this pod |
| minio.service | object | `{"annotations":{},"type":"ClusterIP"}` | Service settings for Minio |
| minio.tolerations | list | `[]` | tolerations for Minio deployment |
| postgres | object | `{"affinity":{},"enabled":true,"image":{"pullPolicy":"IfNotPresent","repository":"postgres","tag":"10.16"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"1000m","memory":"512Mi"},"requests":{"cpu":"10m","memory":"128Mi"}},"service":{"annotations":{},"type":"ClusterIP"},"tolerations":[]}` |  POSTGRES SETTINGS |
| postgres.affinity | object | `{}` | affinity for Postgres deployment |
| postgres.enabled | bool | `true` | - enable or disable Postgres deployment installation |
| postgres.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| postgres.image.repository | string | `"postgres"` | Docker image for Postgres deployment |
| postgres.image.tag | string | `"10.16"` | Docker image tag |
| postgres.nodeSelector | object | `{}` | nodeSelector for Postgres deployment |
| postgres.podAnnotations | object | `{}` | Annotations for Postgres pods |
| postgres.replicaCount | int | `1` | Replicas count for Postgres deployment |
| postgres.resources | object | `{"limits":{"cpu":"1000m","memory":"512Mi"},"requests":{"cpu":"10m","memory":"128Mi"}}` | Default resources requests and limits for Postgres deployment |
| postgres.service | object | `{"annotations":{},"type":"ClusterIP"}` | Service settings for Postgres |
| postgres.tolerations | list | `[]` | tolerations for Postgres deployment |
| pytorchoperator | object | `{"affinity":{},"enabled":true,"image":{"pullPolicy":"IfNotPresent","repository":"gcr.io/kubeflow-images-public/pytorch-operator","tag":"v1.0.0-g047cf0f"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"500m","memory":"1000M"},"requests":{"cpu":"10m","memory":"50M"}},"service":{"annotations":{},"type":"ClusterIP"},"serviceAccountAnnotations":{},"tolerations":[]}` | -------------------- Distributed Pytorch execution using the K8s Pytorch operator |
| pytorchoperator.affinity | object | `{}` | affinity for Pytorchoperator deployment |
| pytorchoperator.enabled | bool | `true` | - enable or disable Pytorchoperator deployment installation |
| pytorchoperator.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| pytorchoperator.image.repository | string | `"gcr.io/kubeflow-images-public/pytorch-operator"` | Docker image for Pytorchoperator |
| pytorchoperator.image.tag | string | `"v1.0.0-g047cf0f"` | Docker image tag |
| pytorchoperator.nodeSelector | object | `{}` | nodeSelector for Pytorchoperator deployment |
| pytorchoperator.podAnnotations | object | `{}` | Annotations for Pytorchoperator pods |
| pytorchoperator.replicaCount | int | `1` | Replicas count for Pytorchoperator deployment |
| pytorchoperator.resources | object | `{"limits":{"cpu":"500m","memory":"1000M"},"requests":{"cpu":"10m","memory":"50M"}}` | Default resources requests and limits for Pytorchoperator |
| pytorchoperator.service | object | `{"annotations":{},"type":"ClusterIP"}` | Service settings for Pytorchoperator |
| pytorchoperator.serviceAccountAnnotations | object | `{}` | Annotations for ServiceAccount attached to Pytorchoperator pods |
| pytorchoperator.tolerations | list | `[]` | tolerations for Pytorchoperator deployment |
| redis | object | `{"affinity":{},"enabled":true,"image":{"pullPolicy":"IfNotPresent","repository":"docker.io/bitnami/redis","tag":"4.0.2-r1"},"nodeSelector":{},"podAnnotations":{},"replicaCount":1,"resources":{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"10m","memory":"50Mi"}},"service":{"annotations":{},"type":"ClusterIP"},"tolerations":[]}` | -------------------------------------------- Sandbox Configuration Sandbox allows to run flyte without any cloud dependencies and can be run even locally on your laptop. This is achieved by replacing cloud service dependencies by k8s local alternatives. These may not be ideal for a high performance setup, but are great to try out flyte ----------------------------------------------- REDIS SETTINGS |
| redis.affinity | object | `{}` | affinity for Redis Statefulset |
| redis.enabled | bool | `true` | - enable or disable Redis Statefulset installation |
| redis.image.pullPolicy | string | `"IfNotPresent"` | Docker image pull policy |
| redis.image.repository | string | `"docker.io/bitnami/redis"` | Docker image for Redis Statefulset |
| redis.image.tag | string | `"4.0.2-r1"` | Docker image tag |
| redis.nodeSelector | object | `{}` | nodeSelector for Redis Statefulset |
| redis.podAnnotations | object | `{}` | Annotations for Redis pods |
| redis.replicaCount | int | `1` | Replicas count for Redis Statefulset |
| redis.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"10m","memory":"50Mi"}}` | Default resources requests and limits for Redis Statefulset |
| redis.resources.limits | object | `{"cpu":"1000m","memory":"1Gi"}` | Limits are the maximum needed resources for this pod. |
| redis.resources.requests | object | `{"cpu":"10m","memory":"50Mi"}` | Requests are the minimum required resources for this pod. |
| redis.service | object | `{"annotations":{},"type":"ClusterIP"}` | Service settings for Redis |
| redis.tolerations | list | `[]` | tolerations for Redis Statefulset |
| sagemaker | object | `{"enabled":false,"plugin_config":{"plugins":{"sagemaker":{"region":"<region>","roleArn":"<arn>"}}}}` | --------------- -- Training on AWS Sagemaker using AWS Sagemaker operator. To actually install the operator, please follow instructions [here](https://github.com/aws/amazon-sagemaker-operator-for-k8s/tree/master/hack/charts/installer/rolebased) Use the config section here to just enable sagemaker plugin in Flyte, after you have installed the operator using the information |
| sparkoperator | object | `{"enabled":true,"image":{"tag":"v1beta2-1.2.0-3.0.0"},"plugin_config":{"plugins":{"spark":{"spark-config-default":[{"spark.hadoop.fs.s3a.aws.credentials.provider":"com.amazonaws.auth.DefaultAWSCredentialsProviderChain"},{"spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version":"2"},{"spark.kubernetes.allocation.batch.size":"50"},{"spark.hadoop.fs.s3a.acl.default":"BucketOwnerFullControl"},{"spark.hadoop.fs.s3n.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3n.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3a.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.multipart.threshold":"536870912"},{"spark.blacklist.enabled":"true"},{"spark.blacklist.timeout":"5m"},{"spark.task.maxfailures":"8"}]}}},"replicaCount":1,"resources":{"limits":{"cpu":"1000m","memory":"500M"},"requests":{"cpu":"10m","memory":"50M"}}}` | ------------------------------------------------------ Optional Plugins -------------------------------------------------------- -- Optional: Spark Plugin using the Spark Operator |
| sparkoperator.enabled | bool | `true` | - enable or disable Sparkoperator deployment installation |
| sparkoperator.image.tag | string | `"v1beta2-1.2.0-3.0.0"` | Docker image for Sparkoperator |
| sparkoperator.plugin_config | object | `{"plugins":{"spark":{"spark-config-default":[{"spark.hadoop.fs.s3a.aws.credentials.provider":"com.amazonaws.auth.DefaultAWSCredentialsProviderChain"},{"spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version":"2"},{"spark.kubernetes.allocation.batch.size":"50"},{"spark.hadoop.fs.s3a.acl.default":"BucketOwnerFullControl"},{"spark.hadoop.fs.s3n.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3n.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3a.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.multipart.threshold":"536870912"},{"spark.blacklist.enabled":"true"},{"spark.blacklist.timeout":"5m"},{"spark.task.maxfailures":"8"}]}}}` | Spark plugin configuration |
| sparkoperator.plugin_config.plugins.spark.spark-config-default | list | `[{"spark.hadoop.fs.s3a.aws.credentials.provider":"com.amazonaws.auth.DefaultAWSCredentialsProviderChain"},{"spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version":"2"},{"spark.kubernetes.allocation.batch.size":"50"},{"spark.hadoop.fs.s3a.acl.default":"BucketOwnerFullControl"},{"spark.hadoop.fs.s3n.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3n.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.impl":"org.apache.hadoop.fs.s3a.S3AFileSystem"},{"spark.hadoop.fs.AbstractFileSystem.s3a.impl":"org.apache.hadoop.fs.s3a.S3A"},{"spark.hadoop.fs.s3a.multipart.threshold":"536870912"},{"spark.blacklist.enabled":"true"},{"spark.blacklist.timeout":"5m"},{"spark.task.maxfailures":"8"}]` | Spark default configuration |
| sparkoperator.plugin_config.plugins.spark.spark-config-default[0] | object | `{"spark.hadoop.fs.s3a.aws.credentials.provider":"com.amazonaws.auth.DefaultAWSCredentialsProviderChain"}` |  it can use the serviceAccount based IAM role or ec2 metadata based. This is more in line with how AWS works |
| sparkoperator.replicaCount | int | `1` | Replicas count for Sparkoperator deployment |
| sparkoperator.resources | object | `{"limits":{"cpu":"1000m","memory":"500M"},"requests":{"cpu":"10m","memory":"50M"}}` | Default resources requests and limits for Sparkoperator |
| storage | object | `{"bucketName":"my-s3-bucket","custom":{},"gcs":null,"s3":{"region":"us-east-1"},"type":"sandbox"}` | --------------------------------------------------- Core dependencies that should be configured for Flyte to work on any platform Specifically 2 - Storage (s3, gcs etc), Production RDBMS - Aurora, CloudSQL etc ------------------------------------------------------ STORAGE SETTINGS |
| storage.bucketName | string | `"my-s3-bucket"` | bucketName defines the storage bucket flyte will use. Required for all types except for sandbox. |
| storage.custom | object | `{}` | GCP project ID. Required for storage type gcs. projectId: -- Settings for storage type custom. See https://github:com/graymeta/stow for supported storage providers/settings. |
| storage.gcs | string | `nil` | settings for storage type gcs |
| storage.s3 | object | `{"region":"us-east-1"}` | settings for storage type s3 |
| storage.type | string | `"sandbox"` | Sets the storage type. Supported values are sandbox, s3, gcs and custom. |
| tf_operator | object | `{"enabled":false}` | -------------------- -- Distributed tensorflow training using the TF-Operator. This can be installed as explained [here](https://github.com/kubeflow/tf-operator) There is no helm chart maintained and hence it might need an independent installation |
| webhook | object | `{"enabled":true,"service":{"annotations":{"projectcontour.io/upstream-protocol.h2c":"grpc"},"type":"ClusterIP"},"serviceAccount":{"annotations":{},"create":true,"imagePullSecrets":{}}}` |  WEBHOOK SETTINGS |
| webhook.enabled | bool | `true` | enable or disable secrets webhook |
| webhook.service | object | `{"annotations":{"projectcontour.io/upstream-protocol.h2c":"grpc"},"type":"ClusterIP"}` | Service settings for the webhook |
| webhook.serviceAccount | object | `{"annotations":{},"create":true,"imagePullSecrets":{}}` | Configuration for service accounts for the webhook |
| webhook.serviceAccount.annotations | object | `{}` | Annotations for ServiceAccount attached to the webhook |
| webhook.serviceAccount.create | bool | `true` | Should a service account be created for the webhook |
| webhook.serviceAccount.imagePullSecrets | object | `{}` | ImapgePullSecrets to automatically assign to the service account |
| workflow_notifications | object | `{"config":{},"enabled":false}` | **Optional Component** Workflow notifications module is an optional dependency. Flyte uses cloud native pub-sub systems to notify users of various events in their workflows |
| workflow_scheduler | object | `{"config":{},"enabled":false}` | -------------------------------------------------------------- Optional Modules Flyte built extensions that enable various additional features in Flyte. All these features are optional, but are critical to run certain features ------------------------------------------------------------------------ -- **Optional Component** Flyte uses a cloud hosted Cron scheduler to run workflows on a schedule. The following module is optional. Without, this module, you will not have scheduled launchplans / workflows. Docs: https://docs.flyte.org/en/latest/howto/enable_and_use_schedules.html#setting-up-scheduled-workflows |
