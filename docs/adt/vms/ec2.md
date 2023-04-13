# AWS EC2

## Overview

To instantiate MiCADO workers on a cloud through EC2 interface, please use the
template below. MiCADO **requires** region_name, image_id and instance_type to
instantiate a VM through *EC2*.

**Terraform** supports provisioning on AWS EC2, and **Occopus** supports
both AWS EC2 and OpenNebula EC2. To use Terraform,
[enable it](/install/cli-install/#enable-terraform) and adjust the interfaces section accordingly.

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.EC2.Compute
  properties:
    region_name: ADD_YOUR_REGION_NAME_HERE (e.g. eu-west-1)
    image_id: ADD_YOUR_ID_HERE (e.g. ami-12345678)
    instance_type: ADD_YOUR_INSTANCE_TYPE_HERE (e.g. t1.small)

  interfaces:
    Occopus:
      create:
        inputs:
          endpoint: ADD_YOUR_ENDPOINT (e.g https://ec2.eu-west-1.amazonaws.com)
```

## Properties

Under the **properties** section of an EC2 virtual machine definition these
inputs are available.:

* **region_name** is the region name within an EC2 cloud (e.g. eu-west-1).
* **image_id** is the image id (e.g. ami-12345678) on your EC2 cloud. Select an
  image containing a base os installation with cloud-init support!
* **instance_type** is the instance type (e.g. t1.small) of your VM to be
  instantiated.
* **key_name** optionally specifies the keypair (e.g. my_ssh_keypair) to be
  deployed on your VM.
* **security_group_ids** optionally specify security settings (you can define
  multiple security groups or just one, but this property must be formatted as
  a list, e.g. [sg-93d46bf7]) of your VM.
* **subnet_id** optionally specifies subnet identifier (e.g. subnet-644e1e13)
  to be attached to the VM.

## Interfaces

Under the **interfaces** section of an EC2 virtual machine definition, the
**endpoint** input is required by Occopus as seen in the example above.

For Terraform the endpoint is discovered automatically based on region.
To customise the endpoint pass the **endpoint** input in interfaces.

```yaml
...
  interfaces:
    Terraform:
      create:
        inputs:
          endpoint: ADD_YOUR_ENDPOINT (e.g https://my-custom-endpoint/api)
```
