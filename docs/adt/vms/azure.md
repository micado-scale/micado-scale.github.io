# Microsoft Azure

## Overview

To instantiate MiCADO workers on a cloud through Azure interface, please
use the template below. Currently, only **Terraform** has support for Azure,
so [Terraform must be enabled](/install/cli-install/#enable-terraform), and the interface must
be set to Terraform as in the example below.

MiCADO supports Windows VM provisioning in Azure. To force a Windows VM,
simply **DO NOT** pass the **public_key** property and **set the image** to
a desired WindowsServer Sku (2016-Datacenter). [Refer to this Sku list](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/cli-ps-findimage#table-of-commonly-used-windows-images)

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.Azure.Compute
  properties:
    resource_group: ADD_YOUR_RG_HERE (e.g. my-test)
    virtual_network: ADD_YOUR_VNET_HERE (e.g. my-test-vnet)
    subnet: ADD_YOUR_SUBNET_HERE (e.g. default)
    network_security_group: ADD_YOUR_NSG_HERE (e.g. my-test-nsg)
    size: ADD_YOUR_ID_HERE (e.g. Standard_B1ms)
    image: ADD_YOUR_IMAGE_HERE (e.g. 18.04.0-LTS or 2016-Datacenter)
    public_key: ADD_YOUR_MINIMUM_2048_KEY_HERE (e.g. ssh-rsa ASHFF...)
    public_ip: [OPTIONAL] BOOLEAN_ENABLE_PUBLIC_IP (e.g. true)

  interfaces:
    Terraform:
      create:
```

## Properties

Under the **properties** section of a Azure virtual machine definition these
inputs are available.:

* **resource_group** specifies the name of the resource group in which
  the VM should exist.
* **virtual_network** specifies the virtual network associated with the VM.
* **subnet** specifies the subnet associated with the VM.
* **network_security_group** specifies the security settings for the VM.
* **vm_size** specifies the size of the VM.
* **image** specifies the name of the image.
* **public_ip [OPTIONAL]** Associate a public IP with the VM.
* **key_data** The public SSH key (minimum 2048-bit) to be associated with
  the instance.
  **Defining this property forces creation of a Linux VM. If it is not**
  **defined, a Windows VM will be created**

## Interfaces

Under the **interfaces** section of a Azure virtual machine definition no
specific inputs are required, but **Terraform: create:** should be present

## Authentication

**Authentication** in Azure is supported by MiCADO in two ways:

- The first is by setting up a
  [Service Principal](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_secret.html)
  and providing the required fields in during
  [cloud credential configuration](/install/cli-install#configure-cloud-credentials).

- The other option is by enabling a
  [System-Assigned Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm#enable-system-assigned-managed-identity-during-creation-of-a-vm)
  on the **MiCADO Control Plane** and then
  [modify access control](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/howto-assign-access-portal#use-rbac-to-assign-a-managed-identity-access-to-another-resource)
  of the **current subscription** to assign the role of **Contributor** to
  the **MiCADO Control Plane**
