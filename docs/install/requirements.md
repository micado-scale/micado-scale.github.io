# Requirements

## Ansible Control Node

MiCADO is deployed by Ansible Playbooks - typically remotely via SSH. You will need
a local device or other remote instance that has SSH access to the new virtual machine
for the MiCADO control plane. The requirements for this Ansible control node are:

- [SSH access](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server#step-3-authenticating-to-your-server-using-ssh-keys) to MiCADO control plane
    - Network access to port 22
    - Private SSH key
- [Python 3.8](https://wiki.python.org/moin/BeginnersGuide/Download) or higher
    - [`pip` is required](https://pip.pypa.io/en/stable/installation/) for standard installation
- [Ansible 2.11](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pip-install) or higher

!!! tip
    A correct version of Ansible is provided by the `micado-client` CLI
    recommended for installation, so installing it separately is usually
    not required.

## MiCADO Control Plane

The MiCADO control plane is a K3s server that runs a number of additional
microservices providing additional functionality. See the minimum requirements
below:

- x86_64
- Ubuntu 20.04 or 22.04
- 2 vCPU
- 4 GB RAM
- 15GB Disk
- TCP Port `22` for installation via SSH
- TCP Port `443` for access to the WebUI
- TCP Ports `6443`, `10250` for cluster operation
- UDP Port `58120` for pod-to-pod communication
- [Python 3.5](https://wiki.python.org/moin/BeginnersGuide/Download) or higher

You must configure [SSH key-based authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) for this VM.

## MiCADO Cloud Workers

MiCADO supports provisoning workers on the following clouds. For best
performance, keep the control plane and workers in the same cloud, region
and availability zone.

??? info "Supported Clouds"
    - Amazon EC2
    - OpenStack Nova
    - Microsoft Azure
    - Google Cloud
    - CloudSigma
    - CloudBroker

Worker requirements will vary based on your specific application requirements,
but the minimum requirements are below:

- x86_64
- Ubuntu 20.04 or 22.04
- 1 vCPU
- 1 GB RAM
- 5 GB Disk
- UDP Port `58120`

## MiCADO Edge Workers

Requirements at the edge are minimal, and are mostly dictated by
[KubeEdge](https://kubeedge.io).

- x86_64 or arm64/aarch64
- Most Linux distributions (tested on Ubuntu & Raspbian)
- TCP Port 22 (SSH) for initial edge setup
