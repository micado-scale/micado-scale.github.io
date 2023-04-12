# API Reference

This page provides a reference for the different available fields on
container, volume, config and other related nodes in the ADT.

## Containers

### Properties

The fields under the **properties** section of the Kubernetes app are a
collection of options specific to all iterations of Docker containers.
The translator understands both Docker-Compose style naming and Kubernetes
style naming, though the Kubernetes style is recommended. You can find
additional information about properties at [this link](https://github.com/jaydesl/TOSCAKubed/blob/master/README.md).
These properties will be translated into Kubernetes manifests on deployment.

Here are the fields available under **properties**

* **name**: name for the container (defaults to the TOSCA node name)
* **command**: override the default command line of the container (*list*)
* **args**: override the default entrypoint of container (*list*)
* **env**: list of required environment variables in format:

  * **name:**
  * **value:**
  * **valueFrom:** for use with ConfigMaps, see below
* **envFrom**: mostly for using ConfigMaps, see below
* **resource:**

  * **requests:**

    * **cpu**: CPU reservation, core components usually require 100m so assume
      900m as a maximum
* **ports**: list of published ports to the host machine, you can specify these
  keywords in the style of a flattened (*Service*, *ServiceSpec* and
  *ServicePort* can all be defined at the same level - `see Kubernetes Service
  <https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#service-v1-core>`__)

  * **targetPort**: the port to target (assumes port if not specified)
  * **port**: the port to publish (assumes targetPort if not specified)
  * **name**: the name of this port in the service (generated if not specified)
  * **protocol**: the protocol for the port (defaults to: TCP)
  * **nodePort**: the port (30000-32767) to expose on the host
    (will create a nodePort Service unless type is explicitly set below)
  * **type**: the type of service for this port (defaults to: ClusterIP
    unless nodePort is defined above)
  * **clusterIP**: the desired (internal) IP (10.0.0.0/24) for this service
    (defaults to next available)
  * **metadata**: service metadata, giving the option to set a name for the
    service. Explicit naming can be used to group different ports together
    (default grouping is by type)
  * **hostPort**: the port on the node host to expose the pod at
  * **containerPort**: the port to target if exposing with hostPort

Environment variables can be loaded in from configuration
data in Kubernetes ConfigMaps. This can be accomplished by using **envFrom:**
with a list of **configMapRef:** to load all data from a ConfigMap into
environment variables as seen
`here <https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables>`__
, or by using **env:** and **valueFrom:**  with **configMapKeyRef:** to load
specific values into environment variables as seen
`here <https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data>`__
.

Alternatively, ConfigMaps can be mounted as volumes as discussed
`here <https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume>`__
, in the same way other volumes are attached to a container, using the
**requirements:** notation below. Also see the examples in **Specification**
**of Configuration Data** below.


### Requirements

Under the **requirements** section you can define the virtual machine
you want to host this particular app, restricting the container to run
**only** on that VM. If you do not provide a host requirement, the container
will run on any possible virtual machine. You can also attach a volume or
ConfigMap to this app - the definition of volumes can be found in the next
section. Requirements takes a list of map objects:

* **host:** name of your virtual machine as defined under node_templates
* **volume:**

  * **node:** name of your volume (or ConfigMap) as defined under
    node_templates
  * **relationship:** **!!**

    * **type:** ``tosca.relationships.AttachesTo``
    * **properties:**

      * **location:** path in container

* **container:** name of a sidecar container defined as a
  ``tosca.nodes.MiCADO.Container.Application.Docker`` type under
  node_templates. The sidecar will share the Kubernetes Pod with
  the main container (the sidecar should not be given an interface)
  **OR** name of an init container defined as a
  ``tosca.nodes.MiCADO.Container.Application.Docker.Init`` type
  under node_templates. The Pod will enter a ready state when
  the Init Container runs to completion and exits cleanly (ie. with
  a zero exit code)

If a relationship is not defined for a volume the
path on container will be the same as the path defined in the volume
(see Specification of Volumes). If no path is defined in the volume,
the path defaults to */etc/micado/volumes* for a Volume or
*/etc/micado/configs* for a ConfigMap

### Interfaces

Under the **interfaces** section you can define orchestrator specific
options, to instruct MiCADO to use Kubernetes, we use the key **Kubernetes**.
Fields under **inputs:** will be translated directly to a Kubernetes manifest
so it is possible to use the full range of properties which Kubernetes offers
as long as field names and syntax follow `the Kubernetes documentation <https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#deployment-v1-apps>`__
If **inputs:** is omitted a set of defaults will be used to create a Deployment

* **create**: *this key tells MiCADO to create a workload*
  *(Deployment/DaemonSet/Job/Pod etc...) for this container*

  * **inputs**: *top-level workload and workload spec options go here...
    two examples, for more see* `translator documentation <https://github.com/jaydesl/TOSCAKubed/blob/master/README.md>`__

    * **kind:** overwrite the workload type (defaults to Deployment)
    * **spec:**

      * **strategy:**

        * **type:** Recreate (kill pods then update instead of RollingUpdate)

### Networking in Kubernetes

Kubernetes networking is inherently different to the approach taken by
Docker/Swarm. This is a complex subject which is worth a
[read here](https://kubernetes.io/docs/concepts/cluster-administration/networking/).
Since every pod gets its own IP, which any pod can by default use to
communicate with any other pod, this means there is no network to
explicitly define. If the **ports** keyword is defined in the definition
above, pods can reach each other over CoreDNS via their hostname (container
name).

## Volumes

Volumes are defined at the same level as virtual machines and containers,
and are then connected to containers using the **requirements:** notation
discussed above in the container spec. Some examples of attaching volumes
will follow.

### Interfaces

Under the **interfaces** section you should define orchestrator specific
options, here we again use the key **Kubernetes:**

* **create**: *this key tells MiCADO to create a persistent volume and claim*

  * **inputs**: persistent volume specific spec options... here are two
    popular examples, see `Kubernetes volumes <https://kubernetes.io/docs/concepts/storage/volumes/>`__ for more

    * **spec:**

      * **nfs:**

        * **server:** IP of NFS server
        * **path:** path on NFS share

      * **hostPath:**

        * **path:** path on host

## Configs

Configuration data (a Kubernetes **ConfigMap**) are to be defined at the same
level as virtual machines, containers and volumes and then loaded into
environment variables, or mounted as volumes in the definition of containers
as discussed in **Specification of the Application**.
Some examples of using configurations will follow at the end of this section.

### Interfaces

Currently MiCADO only supports the definition of configuration
data as Kubernetes ConfigMaps. Under the
**interfaces** section of this type use the key **Kubernetes:**
to instruct MiCADO to create a ConfigMap.

* **create**: *this key tells MiCADO to create a ConfigMap*

  * **inputs**: ConfigMap fields to be overwritten, for more detail see
    `ConfigMap <https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#configmap-v1-core>`__

    * **data:** for UTF-8 byte values
    * **binaryData:** for byte values outside of the UTF-8 range
