# Google Compute Engine

## Overview

To instantiate MiCADO workers on a cloud through Google interface, please use
the template below. Currently, only **Terraform** has support for Google Cloud,
so [Terraform must be enabled](/install/cli-install/#enable-terraform), and the
interface must be set to Terraform as in the example below.

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.GCE.Compute
  properties:
    region: ADD_YOUR_ID_HERE (e.g. us-west1)
    zone: ADD_YOUR_ID_HERE (e.g. us-west1-a)
    project: ADD_YOUR_ID_HERE (e.g. PGCE)
    machine_type: ADD_YOUR_ID_HERE (e.g. n1-standard-2)
    image: ADD_YOUR_ID_HERE (e.g.  ubuntu-os-cloud/ubuntu-1804-lts)
    network: ADD_YOUR_ID_HERE (e.g. default)
    ssh-keys: ADD_YOUR_ID_HERE (e.g. ssh-rsa AAAB3N...)

  interfaces:
    Terraform:
      create:
```

## Properties

Under the **properties** section of a GCE virtual machine definition these
inputs are available.:

* **project** is the project to manage the resources in.
* **image** specifies the image from which to initialize the VM disk.
* **region** is the region that the resources should be created in.
* **machine_type** specifies the type of machine to create.
* **zone** is the zone that the machine should be created in.
* **network** is the network to attach to the instance.
* **ssh-keys** sets the public SSH key to be associated with the instance.

## Interfaces

Under the **interfaces** section of a GCE virtual machine definition no
specific inputs are required, but **Terraform: create:** should be present

## Authentication

**Authentication** in GCE is done using a service account key file in JSON
format. You can manage the key files using the Cloud Console. The steps to
retrieve the key file is as follows :

  * Open the **IAM & Admin** page in the Cloud Console.
  * Click **Select a project**, choose a project, and click **Open**.
  * In the left nav, click **Service accounts**.
  * Find the row of the service account that you want to create a key for.
    In that row, click the **More** button, and then click **Create key**.
  * Select a **Key type** and click **Create**.
