# Virtual Machines

The collection of containers specified in the
[previous section](/adt/containers) is orchestrated by Kubernetes. 
This section introduces how the nodes that make up the
Kubernetes cluster will be configured.

During operation MiCADO will instantiate as many virtual machines with
the parameters defined here as required, scaling as required. MiCADO
currently supports seven different cloud interfaces:

- [OpenStack Nova](openstack.md)
- [EC2](ec2.md)
- [CloudBroker](cloudbroker.md)
- [CloudSigma](cloudsigma.md)
- [Azure](azure.md)
- [Google Cloud](google.md)
- [Oracle Cloud](oracle.md)

MiCADO supports multiple sets of nodes, and specific containers can be
[configured](/adt/containers/#choosing-a-host) to run only on specific nodes.
Multi-cloud support has not been fully tested across all combinations of clouds,
but the underlying K3s provides excellent support for this.

!!! warning

    As with containers, underscores are not permitted in virtual machine names.
    Names should begin and end with an alphanumeric.

The following ports and protocols should be enabled on nodes
acting as MiCADO workers, replacing `application dependent` with ports
your application needs exposed on that host.

|Protocol| Port(s)   |  Service             |
|--------|-----------|----------------------|
| *      |  *        | application dependent|
| UDP    |  51820    | wireguard (Pod-Pod)  |


## Overview

Here is how a virtual machine description might appear inside an ADT.

```yaml
wmin-openstack-worker:
  type: tosca.nodes.MiCADO.Nova.Compute
    properties:
      flavor_id: t2.large
      ...
      context:
        cloud_config: |
          runcmd:
          - curl http://example.com || echo "curl failed" > status.log
    capabilities:
      host:
        properties:
          num_cpus: 2
          mem_size: 4 GB
      os:
        properties:
          distribution: ubuntu
          version: 18.04
    interfaces:
      Occopus:
        create:
          inputs:
            endpoint: https://wmin-cloud.ac.uk/api/v3
```

### Properties 

The **properties** section is **REQUIRED** and contains the necessary
properties to provision the virtual machine and vary from cloud to cloud.
Properties for each cloud will vary and are detailed in a separate section
for each supported cloud.

#### Cloud Contextualisation

  It is possible to provide custom configuration of the deployed nodes via
  [cloud-init scripts](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).
  MiCADO relies on a cloud-init config to join nodes to the
  cluster, so it is recommended to only add to the default config, except
  for certain cases.

  The **context** key is supported by all the cloud compute node definitions
  below. New cloud-init configurations should be defined in **cloud_config**
  and *one* of **append**, **insert** or **overwrite** can be set to
  change the behaviour of how the default cloud-init config is affected.

  - Setting **append** to true will add the newly defined configurations
    to the end of the default cloud-init config. This is the default.
  - Setting **insert** to true will add the newly defined configurations
    to the start of the default cloud-init config, before the MiCADO Worker
    is fully initialised
  - Setting **overwrite** to true will overwrite the default cloud-init config.

### Capabilities (Optional)

The **capabilities** sections for all virtual machine definitions that follow
are identical and are **ENTIRELY OPTIONAL**. They are ommited in the
cloud-specific examples below. They are filled with the following metadata to
support human readability:

* **num_cpus** under *host* is an integer specifying number of CPUs for
  the instance type
* **mem_size** under *host* is a readable string with unit specifying RAM of
  the instance type
* **type** under *os* is a readable string specifying the operating system
  type of the image
* **distribution** under *os* is a readable string specifying the OS distro
  of the image
* **version** under *os* is a readable string specifying the OS version of
  the image

### Interfaces

The **interfaces** section of all virtual machine definitions that follow
are **REQUIRED**, and allow you to provide orchestrator specific inputs, in
the examples we use either **Occopus** or **Terraform** based on suitability.

* **create**: *this key tells MiCADO to create the VM using Occopus/Terraform*

  * **inputs**: Extra settings to pass to Occopus or Terraform

    * **endpoint:** the endpoint API of the cloud (always required for
      Occopus, sometimes required for Terraform)


## Pre-defined Types

Through abstraction, it is possible to reference a
pre-defined type and simplify the description of a virtual machine.

!!! example
    Here we use the `.Occo.small` extension of the `CloudSigma.Compute`
    parent node. Notice how we don't specify the VM size, and completely
    omit the interfaces section.

    ```yaml
    wmin-cloudsigma-worker:
      type: tosca.nodes.MiCADO.CloudSigma.Compute.Occo.small
        properties:
          vnc_password: secret
          libdrive_id: 87ce928e-e0bc-4cab-9502-514e523783e3
          public_key_id: d7c0f1ee-40df-4029-8d95-ec35b34dae1e
          nics:
          - firewall_policy: fd97e326-83c8-44d8-90f7-0a19110f3c9d
            ip_v4_conf:
              conf: dhcp
    ```

Currently MiCADO supports these additional types for CloudSigma, but there
is no limit to how many can be authored!

* **tosca.nodes.MiCADO.EC2.Compute.Terra** -
  Orchestrates with Terraform on eu-west-2, overwrite region_name
  under **properties** to change region
* **tosca.nodes.MiCADO.CloudSigma.Compute.Occo** -
  Automatically orchestrates on Zurich with Occopus. There is no need to
  define further fields under **interfaces:** but Zurich can be changed
  by overwriting **endpoint** under **properties:**
* **tosca.nodes.MiCADO.CloudSigma.Compute.Occo.small** -
  As above but creates a 2GHz/2GB node by default
* **tosca.nodes.MiCADO.CloudSigma.Compute.Occo.big** -
  As above but creates a 4GHz/4GB node by default
* **tosca.nodes.MiCADO.CloudSigma.Compute.Occo.small.NFS** -
  As *small* above but installs NFS dependencies by default
