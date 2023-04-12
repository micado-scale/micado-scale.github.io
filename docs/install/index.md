# Installation

To deploy a MiCADO cluster, you need a virtual machine to be set up as the
MiCADO control plane. We deploy a standard Kubernetes distribution (K3s)
and a number of MiCADO-specific compoenents.

Preparation of the control
plane is facilitated by Ansible. If you are familiar with Ansible Playbooks,
download the latest release and configure the relevant files before deployment.

For those new to Ansible, we provide a small command-line tool to ease
deployment of the MiCADO control plane.