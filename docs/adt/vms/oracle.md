# OCI

## Overview

To instantiate MiCADO workers on a cloud through Oracle interface, please use
the template below. Currently, only **Terraform** has support for Oracle,
so Terraform must be enabled as in :ref:`customize`, and the interface must
be set to Terraform as in the example below under ``context``.

**Note** that OCI's Ubuntu VM images feature a number of strict ``iptables``
rules, which will restrict normal communnication between worker nodes and the
MiCADO Master. To resolve this issue, it is important to include the VM
contextualisation commands that can be seen in the example below.


```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.OCI.Compute
  properties:
    region: <REGION_NAME> (e.g. uk-london-1)
    availability_domain: <AVAILABILITY_DOMAIN> (e.g. lVvK:UK-LONDON-1-AD-1)
    compartment_id: <COMPARTMENT_OCID> (e.g ocid1.tenancy.oc1..aaa)
    shape: <VM_TYPE_NAME> (e.g. VM.Standard.E2.1)
    source_id: <VM_IMAGE_OCID> (e.g ocid1.image.oc1.uk-london-1.aaa)
    subnet_id: <SUBNET_OCID> (e.g ocid1.subnet.oc1.uk-london-1.aaa)
    network_security_group: <NETWORK_SECURITY_GROUP_OCID> (e.g ocid1.networksecuritygroup.oc1.uk-london-1.aaa)
    ssh_keys: ADD_YOUR_ID_HERE (e.g. ssh-rsa AAAB3N...)
    context:
      insert: true
      cloud_config: |
        runcmd:
        - iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
        - iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited

  interfaces:
    Terraform:
      create:
```

## Properties

Under the **properties** section of a OCI virtual machine definition these
inputs are available.:

* **availability_domain** is the availability domain of the instance.
* **source_id** specifies the OCID of an image from which to initialize the
  VM disk.
* **region** is the region that the resources should be created in.
* **shape** specifies the type of machine to create.
* **compartment_id** is the OCID of the compartment.
* **subnet_id** is the OCID of the subnet to create the VNIC in.
* **network_security_group** specifies the OCID of the network security
  settings for the VM.
* **ssh_keys** sets the public SSH key to be associated with the instance.

## Interfaces

Under the **interfaces** section of an OCI virtual machine definition no
specific inputs are required, but **Terraform: create:** should be present.

## Authentication

**Authentication** in OCI is supported by MiCADO in two ways:

- The first is by setting up an [Instance Principal](https://www.terraform.io/docs/providers/oci/index.html)
  based authentication on the **MiCADO Control Plane** by creating suitable [Dynamic Group and Policies](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm)
  associated with it.

- The other option is by enabling an
  [API Key](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#five)
  based authentication on the **MiCADO Control Plane** and providing the required
  fields during [cloud credential configuration](/install/cli-install#configure-cloud-credentials).
