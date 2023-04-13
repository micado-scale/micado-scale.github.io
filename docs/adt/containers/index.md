# Describing Containers

- The Basics (this page)
- [**Services and container communication**](services.md)
- [**Exposing containers externally**](expose.md)
- [**Volumes and Configs**](volumes.md)
- [**Multi-Container Pods (Sidecars)**](sidecars.md)
- [**Special MiCADO/Kubernetes Types**](custom.md)


## Getting Started

This page will get you started with your first container. The links above will point you
to more specific information, once you are ready for it.

### Properties and Interfaces

[In case you missed it](/adt/#a-closer-look), there is a clear distinction
between `properties` and `interfaces`. For containers specifically:

* We use `properties` to define options and parameters that are general
  enough that they could be applied to _any_ Docker container, regardless of the orchestrator.
* We use `inputs` under `interfaces` to indicate that we want to overwrite
  the fields that one would find in a generated Kubernetes manifest

!!! note
    Unless you've been explicit, MiCADO will try to orchestrate your
    container as a Kubernetes **Deployment**.

### The bare minimum

At a minimum, you'll need a container image in DockerHub or another container registry
that you want to use. Here's a minimum working example:

```yaml
app-container:
  type: tosca.nodes.MiCADO.Container.Application.Docker
  properties:
    image: nginx
  interfaces:
    Kubernetes:
      create:
```
> If you're interested, see how the generated Kubernetes manifest will look by opening the box below.

??? Example "Kubernetes Manifest"

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-container
      labels:
        app.kubernetes.io/name: app-container
        app.kubernetes.io/instance: mwe
        app.kubernetes.io/managed-by: micado
        app.kubernetes.io/version: latest
    spec:
      template:
        metadata:
          labels:
            app.kubernetes.io/name: app-container
            app.kubernetes.io/instance: mwe
            app.kubernetes.io/managed-by: micado
            app.kubernetes.io/version: latest
        spec:
          containers:
          - image: nginx
            name: app-container
      selector:
        matchLabels:
          app.kubernetes.io/name: app-container
          app.kubernetes.io/instance: mwe
          app.kubernetes.io/managed-by: micado
          app.kubernetes.io/version: latest
    ```

For those familiar with Kubernetes, this mounts a single **container** (`nginx:latest`)
inside a **Pod** owned by a **Deployment** using a set of labels based on
[Kubernetes recommendations](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/#labels).
For those not familiar - this minimum working example creates a scalable unit of your specified container.

!!! warning

    Note that because Kubernetes does not support underscores in resources names,
    neither do we.

### Choosing a host

Often, you'll want to be sure that your container runs on a specific node
(virtual machine) within your deployment. You can accomplish this by
referencing the virtual machine node in the **requirements** section of
your container description, like so:

```yaml
vm-node:
  type: tosca.nodes.MiCADO.EC2.Compute
  ...(truncated)...

app-container:
  type: tosca.nodes.MiCADO.Container.Application.Docker
  properties:
    image: nginx
  requirements:
  - host: vm-node
  interfaces:
    Kubernetes:
      create:
```

> In Kubernetes-speak, this adds a NodeAffinity to the generated manifest.
> See the box below if you're interested in how it would look.

??? Example "Kubernetes manifest"

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-container
      labels:
        app.kubernetes.io/name: app-container
        app.kubernetes.io/instance: affinity
        app.kubernetes.io/managed-by: micado
        app.kubernetes.io/version: latest
    spec:
      template:
        metadata:
          labels:
            app.kubernetes.io/name: app-container
            app.kubernetes.io/instance: affinity
            app.kubernetes.io/managed-by: micado
            app.kubernetes.io/version: latest
        spec:
          containers:
          - image: nginx      
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  - key: micado.eu/node_type
                    operator: In
                    values:
                    - vm-node
      selector:
        matchLabels:
          app.kubernetes.io/name: app-container
          app.kubernetes.io/instance: affinity
          app.kubernetes.io/managed-by: micado
          app.kubernetes.io/version: latest
    ```

## More properties

You will likely need to provide more customisation for your container than simply specifying an image. The available list of properties is available in the [API Reference](specification.md) section.

### Switching it up

If you need something more specific than the basic **Deployment** workload, you'll have to say so using `inputs`. Here's how it would look:

```yaml
app-container:
  type: tosca.nodes.MiCADO.Container.Application.Docker
  properties:
    image: nginx
  interfaces:
    Kubernetes:
      create:
        inputs:
          kind: DaemonSet
```

> The DaemonSet workload creates a replica of the container on each active node, like a _global_ service in Docker Swarm

We support all of the [Workload Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/), though to get certain workloads to validate, you'll have to provide further parameters under `inputs`.

### More inputs

If you're comfortable with Kubernetes manifests, you can fully customise the final generated manifest by using `inputs`. There's no definitive list of available options - you are only restricted by what's in the [Kubernetes API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#-strong-workloads-apis-strong-). Here's an example with **StatefulSets**:

```yaml
app-stateful:
  type: tosca.nodes.MiCADO.Container.Application.Docker
  properties:
    image: nginx
  interfaces:
    Kubernetes:
      create:
        inputs:
          kind: StatefulSet
          metadata:
            labels:
              interesting_label: info
          spec:
            serviceName: app-stateful
            updateStrategy:
              type: RollingUpdate
            template:
              spec:
                hostNetwork: True
```
> Here we're adding lots of Kubernetes specific customisation, right down to the
> level of the **PodSpec**. See the generated manifest in the box below. Notice that
> the container we've described using `properties` is still at the core of this StatefulSet.

??? example "Kubernetes manifest"

    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      labels:
        interesting_label: info
        app.kubernetes.io/name: app-stateful
        app.kubernetes.io/instance: stateful-ex
        app.kubernetes.io/managed-by: micado
        app.kubernetes.io/version: latest
      name: app-stateful
    spec:
      serviceName: app-stateful
      updateStrategy:
        type: RollingUpdate
      template:
        spec:
          containers:
          - image: nginx
            name: app-stateful
          hostNetwork: true
        metadata:
          labels:
            app.kubernetes.io/name: app-stateful
            app.kubernetes.io/instance: stateful-ex
            app.kubernetes.io/managed-by: micado
            app.kubernetes.io/version: latest
      selector:
        matchLabels:
          app.kubernetes.io/name: app-stateful
          app.kubernetes.io/instance: stateful-ex
          app.kubernetes.io/managed-by: micado
          app.kubernetes.io/version: latest
    ```

## Next up: [Services and Container Communication](services.md)
