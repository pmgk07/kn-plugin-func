# Project Configuration with `func.yaml`

The `func.yaml` file contains configuration information for your function
project. Generally, these values are used when you execute a `func` CLI
command. For example, when `func build` is run, the CLI uses the value for
the `builder` field. In many cases, these values may be overridden by
command line flags or environment variables. For more information about
overriding these values, consult the [Commands](command.md) document.

Many of the fields are generated for you when you create, build and deploy
your function. However there are a few that you may use to tweak things
such as the function name, and the image name.

## Fields

The following fields are used in `func.yaml`.

### `builder`

Specifies the buildpack builder image to use when building the function.
In most cases, this value should not be changed.

### `builders`

Some function runtimes may be built in multiple ways. For example, a Quarkus
function may be built for the JVM, or as a native binary. The `builders`
field will contain all of the available builders for a given runtime. Although
it's typically unnecessary to modify the `builder` field, using values from
`builders` is OK.

### `envs`

The `envs` field allows you to set environment variables that will be
available to your function at runtime. 
1. Environment variable can be set directly from a value
2. Environment variable can be set from a local environment value. Eg. `'{{ env:LOCAL_ENV_VALUE }}'`, for more details see [Local Environment Variables section](#local-environment-variables).
3. Environment variable can be set from a key in a Kubernetes Secret or ConfigMap. This Secret/ConfigMap needs to be created before it is referenced in a function. Eg. `'{{ secret:mysecret:key }}'` where `mysecret` is the name of the Secret and `key` is the referenced key; or `{{ configMap:myconfigmap:key }}` where `myconfigmap` is the name of the ConfigMap and `key` is the referenced key.
4. All key-value pairs from a Kubernetes Secret or ConfigMap will be set as environment variables. This Secret/ConfigMap needs to be created before it is referenced in a function. Eg. `'{{ secret:mysecret2 }}'` where `mysecret2` is the name of the Secret: or `{{ configMap:myconfigmap }}` where `myconfigmap` is the name of the ConfigMap.

```yaml
envs:
- name: EXAMPLE1                            # (1) env variable directly from a value
  value: value
- name: EXAMPLE2                            # (2) env variable from a local environment value
  value: '{{ env:LOCAL_ENV_VALUE }}'
- name: EXAMPLE3                            # (3) env variable from a key in Secret
  value: '{{ secret:mysecret:key }}'
- name: EXAMPLE4                            # (3) env variable from a key in ConfigMap
  value: '{{ configMap:myconfigmap:key }}'
- value: '{{ secret:mysecret2 }}'           # (4) all key-value pairs in Secret as env variables
- value: '{{ configMap:myconfigmap2 }}'     # (4) all key-value pairs in ConfigMap as env variables
```

### `image`

This is the image name for your function after it has been built. This field
may be modified and `func` will create your image with the new name the next
time you run `kn func build` or `kn func deploy`.

### `imageDigest`

This is the `sha256` hash of the image manifest when it is deployed. This value
should not be modified.

### `labels`

The `labels` field allows you to set labels on a deployed function. Labels can be set
directly from a value or from a local environment value. Eg. `'{{ env:USER }}'`, for more details see [Local Environment Variables section](#local-environment-variables).

```yaml
labels:
- key: role                                # (1) label directly from a value
  value: backend
- key: author                              # (2) label from a local environment value
  value: '{{ env:USER }}'
```

### `name`

The name of your function. This value will be used as the name for your service
when it is deployed. This value may be changed to rename the function on
subsequent deployments.

### `namespace`

The Kubernetes namespace where your function will be deployed.

### `options`
Options allows you to set specific configuration for the deployed function, allowing you to tweak Knative Service options related to autoscaling and other properties. If these options are not set, the Knative defaults will be used. 
- `scale`
  - `min`: Minimum number of replicas. Must me non-negative integer, default is 0. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/scale-bounds/#lower-bound).
  - `max`: Maximum number of replicas. Must me non-negative integer, default is 0 - meaning no limit. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/scale-bounds/#upper-bound).
  - `metric`: Defines which metric type is watched by the Autoscaler. Could be `concurrency` (default) or `rps`. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/autoscaling-metrics/).
  - `target`: Recommendation for when to scale up based on the concurrent number of incoming request. Defaults to `options.resources.limits.concurrency` when given. Can be float value greater than 0.01, default is 100. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/concurrency/#soft-limit).
  - `utilization`: Percentage of concurrent requests utilization before scaling up. Can be float value between 1 and 100, default is 70. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/concurrency/#target-utilization).
- `resources`
  - `requests` 
    - `cpu`: A CPU resource request for the container with deployed function. See related [Kubernetes docs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits).
    - `memory`: A memory resource request for the container with deployed function. See related [Kubernetes docs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits).
  - `limits` 
    - `cpu`: A CPU resource limit for the container with deployed function. See related [Kubernetes docs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits).
    - `memory`: A memory resource limit for the container with deployed function. See related [Kubernetes docs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits).
    - `concurrency`: Hard Limit of concurrent requests to be processed by a single replica. Can be integer value greater than or equal to 0, default is 0 - meaning no limit. See related [Knative docs](https://knative.dev/docs/serving/autoscaling/concurrency/#hard-limit).

```yaml
options:
  scale:
    min: 0
    max: 10
    metric: concurrency
    target: 75
    utilization: 75
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 1000m
      memory: 256Mi
      concurrency: 100
```

### `runtime`

The language runtime for your function. For example `python`.

### `template`

The source code template tailored for the invocation event that triggers
your function. For example `http` for plain HTTP requests, `event` for
CloudEvent triggered functions.

### `volumes`
Kubernetes Secrets or ConfigMaps can be mounted to the function as a Kubernetes Volume accessible under specified path. Below you can see an example how to mount the Secret `mysecret` to the path `/workspace/secret` and the ConfigMap `myconfigmap` to the path `/workspace/configmap`. This Secret/ConfigMap needs to be created before it is referenced in a function.

```yaml
volumes:
- secret: mysecret
  path: /workspace/secret
- configMap: myconfigmap
  path: /workspace/configmap
```


## Local Environment Variables

Any of the fields in `func.yaml` may contain a reference to an environment
variable available in the local environment. For example, if I would like
to avoid storing sensitive information such as an API key in my function
configuration, I may have this value set from the local environment. To do
this, prefix the local environment variable with `{{` and `}}` and prefix
the name with `env:`. For example:

```yaml
envs:
- name: API_KEY
  value: '{{ env:API_KEY }}'
```
