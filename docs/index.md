# MiCADO

![image](https://user-images.githubusercontent.com/35102795/231382541-e0b3cfcc-258d-4c04-874a-0d10d1056151.png)

Originally developed in Project [COLA](https://project-cola.eu/). MiCADO is currently used across multiple European projects. Development is ongoing at [this github repository](https://github.com/micado-scale).

## Intro

MiCADO is a generic execution and auto-scaling framework for OCI containers, orchestrated by Kubernetes. It supports autoscaling at two levels. At virtual machine (VM) level, a built-in Kubernetes cluster is dynamically extended or reduced by adding/removing cloud virtual machines. At container level, the number of replicas tied to a specific Kubernetes Deployment can be increased/decreased.

MiCADO expects a TOSCA-based Application Description Template (ADT) to be submitted containing these sections:

1. Definition of the individual applications making up a Kubernetes Deployment,
2. Specification of the virtual machine and
3. (optional) Implementation of policies for scaling and monitoring VM and/or application.

The format of the ADT for MiCADO is detailed later.

To use MiCADO, first the MiCADO core services must be deployed on a virtual machine (the MiCADO Control Plane), usually by an Ansible playbook. The MiCADO Control Plane is configured as a Kubernetes Control Plane and installs a container runtime, a cloud orchestrator (Occopus or Terraform), Prometheus, Policy Keeper and TOSCASubmitter to realise the control loops. During operation MiCADO workers (realised on dynamically provisioned VMs or Edges) are instantiated on demand which deploy Prometheus Node Exporter and expose the metrics-server so Grafana can present metrics. The newly instantiated MiCADO workers join the Kubernetes cluster managed by the MiCADO Control Plane.

