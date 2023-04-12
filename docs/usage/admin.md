# Cluster Admin

Some common cluster admin functionalities are described below.

## Update Cloud & Registry Logins


It is possible to modify the cloud and registry login credentials of
an existing MiCADO cluster. Simply make the necessary changes to the
appropriate credentials file (using `micado config`) and then run the
following CLI command:

```
micado deploy --update-auth
```

## Inspect MiCADO Components

You can inspect the status and logs of MiCADO components via the
[Kubernetes Dashboard](dashboard.md#kubernetes-dashboard). Navigate to them
by changing the **namespace** to `micado-system` or `micado-worker` and then
viewing details of specific components in the **Pods** section

You can also SSH into the MiCADO Control Plane and check the logs at any point
after MiCADO is succesfully deployed. All logs are kept under `/var/log/micado`
and are organised by components. Scaling decisions, for example, can be inspected
under `/var/log/micado/policykeeper`


### Accessing user-defined services

In case your application contains a container exposing a service, you will have to ensure the following to access it.

* First set **nodePort: xxxxx** (where xxxxx is a port in range 30000-32767) in the **properties: ports:** TOSCA description of your docker container. More information on this in the ADT
* The container will be accessible at *<IP>:<port>* . Both, the IP and the port values can be extracted from the Kubernetes Dashboard (in case you forget it). The **IP** can be found under *Nodes > my_micado_vm > Addresses* menu, while the **port** can be found under *Discovery and load balancing > Services > my_app > Internal endpoints* menu.
