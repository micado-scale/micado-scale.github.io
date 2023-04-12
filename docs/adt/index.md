# Application Description Template


MiCADO executes applications described by Application Description Templates (ADT). The
ADT is based on the
[TOSCA Specification](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/TOSCA-Simple-Profile-YAML-v1.3.pdf)
and is described in detail in this section. Some familiarity with YAML is helpful.

An ADT describes the following aspects of an application, which are documented individually:

- [Containers](containers.md)
- Volumes
- Configs
- [Virtual Machines](vms.md)
- [Monitoring](monitoring.md)
- [Scaling](scaling.md)
- [Networking](networking.md)
- [Secrets](secrets.md)

## Overview

* Application Description Templates are the domain specific language of MiCADO.

* They describe the container, virtual machine and scaling/security policies that make up an application deployment.

* These different sections of the ADT get translated and orchestrated by the relevant underlying tools in MiCADO.

### A quick look...

ADTs are based on TOSCA, written in YAML and you can start writing one like this:

```yaml
tosca_definitions_version: tosca_simple_yaml_1_3

imports:
  - https://github.com/micado-scale/tosca/main/micado_types.yaml

description: ADT for stressng on EC2
```

Next up, the `topology_template` is where you describe your containers, virtual machines and policies, beginning with **container** and **virtual machines** under `node_templates`  like so:

```yaml
topology_template:
  node_templates:
  
    app-container:
      type: tosca.nodes.MiCADO.Container.Application.Docker
      properties:
        image: uowcpc/nginx:v1.2
      interfaces:
        Kubernetes:
          create:
            inputs:
              kind: Deployment


    worker-virtualmachine:
      type: tosca.nodes.MiCADO.EC2.Compute
      properties:
        instance_type: t2.small
      interfaces:
        Terraform:
          create:
      
```

### A closer look...

* Looking above, you can see we use a specific `type` to let our adaptors know what they are looking at.

* You can think of `properties` as providing the options and parameters specific to the **basic resource** that you want to orchestate (above, the **Docker container**, and the **EC2 compute instance**).
  
* The `interfaces` section, on the other hand, specifies the **orchestrator** that should be used to manage the defined resouce, with the option to pass in options and parameters for that tool (above, **Kubernetes** and **Terraform**).



## Main sections

The main sections of an ADT are documented below.

`tosca_definitions_version`

:   *string.* **Default:** `tosca_simple_yaml_1_3`. The version of TOSCA to work with. v1.3 by default.

`imports`

:   *string[ ].* List of urls pointing to custom TOSCA types. The default url points to the
    custom types defined for MiCADO. Please, do not modify this url.


`topology_template`

:   *map.* The body of the application description.

    `node_templates`

    :   *map.* Definitions of application containers and auxiliary components
        such as volumes and virtual machines.

    `policies`

    :   *map.* Scaling & metric policies.

`types`

:   This section is used to optionally define additional detailed types which
    can be referenced in the **topology_template** section to benefit from
    abstraction. Under **policy_types:** for example, complex scaling logic
    can be defined here, then referenced in the **policies** section above
