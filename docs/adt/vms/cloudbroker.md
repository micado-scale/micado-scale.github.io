# CloudBroker

## Overview

To instantiate MiCADO workers on CloudBroker, please use the template below.
MiCADO **requires** deployment_id and instance_type_id to instantiate a VM on
*CloudBroker*.

Currently, only **Occopus** has support for CloudBroker, so
[Occopus must be enabled](/install/cli-install/#enable-occopus)
and the interface must be set to Occopus as in the example below.

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.CloudBroker.Compute
    properties:
      deployment_id: ADD_YOUR_ID_HERE (e.g. e7491688-599d-4344-95ef-aff79a60890e)
      instance_type_id: ADD_YOUR_ID_HERE (e.g. 9b2028be-9287-4bf6-bbfe-bcbc92f065c0)
      key_pair_id: ADD_YOUR_ID_HERE (e.g. d865f75f-d32b-4444-9fbb-3332bcedeb75)
      opened_port: ADD_YOUR_PORTS_HERE (e.g. '22,2377,7946,8300,8301,8302,8500,8600,9100,9200,4789')

    interfaces:
      Occopus:
        create:
          inputs:
            endpoint: ADD_YOUR_ENDPOINT (e.g https://cola-prototype.cloudbroker.com )
```

## Properties

Under the **properties** section of a CloudBroker virtual machine definition
these inputs are available.:

* **deployment_id** is the id of a preregistered deployment in CloudBroker
  referring to a cloud, image, region, etc. Make sure the image contains a
  base OS (preferably Ubuntu) installation with cloud-init support! The id is
  the UUID of the deployment which can be seen in the address bar of your
  browser when inspecting the details of the deployment.
* **instance_type_id** is the id of a preregistered instance type in
  CloudBroker referring to the capacity of the virtual machine to be deployed.
  The id is the UUID of the instance type which can be seen in the address bar
  of your browser when inspecting the details of the instance type.
* **key_pair_id** is the id of a preregistered ssh public key in CloudBroker
  which will be deployed on the virtual machine. The id is the UUID of the key
  pair which can be seen in the address bar of your browser when inspecting the
  details of the key pair.
* **opened_port** is one or more ports to be opened to the world. This is a
  string containing numbers separated by a comma.

## Interfaces

Under the **interfaces** section of a CloudBroker virtual machine definition,
remember to pass the endpoint of the CloudBroker deployment you are using.
