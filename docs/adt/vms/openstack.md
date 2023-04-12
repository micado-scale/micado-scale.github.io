
# OpenStack (Nova)

## Overview

To instantiate MiCADO workers on a cloud through Nova interface, please use the
template below. MiCADO **requires** image_id, flavor_name, project_id and
network_id to instantiate a VM through *Nova*.

Both **Occopus and Terraform** support Nova provisioning. To use Terraform,
enable it as described in :ref:`customize` and adjust the interfaces section
accordingly.

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.Nova.Compute
  properties:
    image_id: ADD_YOUR_ID_HERE (e.g. d4f4e496-031a-4f49-b034-f8dafe28e01c)
    flavor_name: ADD_YOUR_ID_HERE (e.g. 3)
    project_id: ADD_YOUR_ID_HERE (e.g. a678d20e71cb4b9f812a31e5f3eb63b0)
    network_id: ADD_YOUR_ID_HERE (e.g. 3fd4c62d-5fbe-4bd9-9a9f-c161dabeefde)
    key_name: ADD_YOUR_KEY_HERE (e.g. keyname)
    security_groups:
      - ADD_YOUR_ID_HERE (e.g. d509348f-21f1-4723-9475-0cf749e05c33)

  interfaces:
    Occopus:
      create:
        inputs:
          endpoint: ADD_YOUR_ENDPOINT (e.g https://sztaki.cloud.mta.hu:5000/v3)
```

## Properties

Under the **properties** section of a Nova virtual machine definition these
inputs are available.:

* **project_id** is the id of project you would like to use on your target
  Nova cloud.
* **image_id** is the image id on your Nova cloud. Select an image containing
  a base os installation with cloud-init support!
* **flavor_name** is the id of the desired flavor for the VM.
* **tenant_name** is the name of the Tenant or Project to login with.
* **user_domain_name** is the domain name where the user is located.
* **availability_zone** is the availability zone in which to create the VM.
* **server_name** optionally defines the hostname of VM (e.g.:”helloworld”).
* **key_name** optionally sets the name of the keypair to be associated to the
  instance. Keypair name must be defined on the target nova cloud before
  launching the VM.
* **security_groups** optionally specify security settings (you can define
  multiple security groups in the form of a **list**) for your VM.
* **network_id** is the id of the network you would like to use on your target
  Nova cloud.
* **floating_ip_pool** (Terraform only) is a string specifying the pool of floating
  IPs that this instance should be assigned a random available floating IP from. If
  this property is not specified, the instance will not be assigned a floating IP.
* **floating_ip** (Terraform only) is a string specifying the specific floating IP
  from the above specified pool that this instance should have assigned to it. This
  property should not be used with instances that may scale out to more than one replica.
* **config_drive** (Terraform only) is a boolean to enable use of a configuration
  drive for metadata storage.

## Interfaces

Under the **interfaces** section of a Nova virtual machine definition, the
**endpoint** input (v3 Identity service) is required as seen in the
example above.

For Terraform the endpoint should also be passed as **endpoint**  in inputs.
Depending on the configuration of the OpenStack cluster, it may be necessary
to provide **network_name** in addition to the ID.

```yaml
...
  interfaces:
    Terraform:
      create:
        inputs:
          endpoint: ADD_YOUR_ENDPOINT (e.g https://sztaki.cloud.mta.hu:5000/v3)
          network_name: ADD_YOUR_NETWORK_NAME (e.g mynet-default)
```

## Authentication

**Authentication** in OpenStack is supported by MiCADO in three ways, by specifying the
appropriate fields during [cloud credential configuration](/install/cli-install#configure-cloud-credentials).

- The default method is authenticating with the same credentials
  used to access the OpenStack WebUI by providing
  the **username** and **password** fields during
  configuration.

- Another option is with
  [Application Credentials](https://docs.openstack.org/keystone/queens/user/application_credentials.html)
  For this method, provide **application_credential_id** and
  **applicaiton_credential_secret** when configuring.
  If these fields are filled, **username** and **password** will be
  ignored.

- A third option is with [OpenID Connect](https://openid.net/connect/)
  for which the URL of the OpenID provider (**identity_provider**) and
  a valid **access_token** are required. When providing a literal access
  token is not practical (for example
  with short-lived access tokens), MiCADO supports automatically
  [refreshing access tokens](https://openid.net/specs/openid-connect-core-1_0.html#RefreshTokens)
  First, complete the `openid` section under `pre-authentication` with a
  **url**, **client_id**, **client_secret** and valid **refresh_token**.
  Then, for the value of **access_token** use the following value: `*OPENID`.
