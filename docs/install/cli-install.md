# CLI Install

For a straightforward approach to installing MiCADO on remote machines,
especially for those unfamiliar with Ansible, we recommend using
the command-line interface provided by `micado-client`.

## Pre-requisites

Provision or create a virtual machine with the
[following specification](requirements.md#micado-control-plane).

Prepare a Linux-based system with
[Python, pip and SSH access](requirements.md#ansible-control-node)
to the above virtual machine.<br>This could be a local device.

## Install micado-client

Use the Python package manager to install the latest
[MiCADO-Client](https://github.com/micado-scale/micado-client), a Python
library and command-line interface (CLI) for building and interacting
with a MiCADO cluster.

```bash
pip install micado-client
```

## Collect Playbooks and configuration

The CLI will create a new directory, or populate an empty directory
with Ansible Playbooks and configuration files for deploying MiCADO.
Name this directory as you please.

!!! tip
    If you are a sysadmin responsible for deploying multiple MiCADO
    clusters, you might name the directory according to the specific
    user or cloud you are preparing MiCADO for.

By default, you will get the latest version of MiCADO. Ask for a
specific version with the `‑‑version` flag.


=== "latest"
    ```bash
    micado init jay_aws_micado
    cd jay_aws_micado
    ```

=== "v0.12.0"
    ```bash
    micado init jay_aws_micado --version v0.12.0 
    cd jay_aws_micado
    ```

## Specify the configuration

### Configure Host List

This command will open Ansible's inventory file in your preferred editor.
The Ansible inventory points at the hosts to configure. You must provide
SSH authentication details for the MiCADO Control Plane VM.

```bash
micado config hosts
```

:   **For example,** If you normally SSH to the VM with:<br>
    &emsp;`ssh ubuntu@123.456.78.90 -i /path/to/key`

    Then your inventory file should read as below:

    ```yaml
    all:
      hosts:
        micado:
          ansible_host: 123.456.78.90
          ansible_connection: ssh
          ansible_user: ubuntu
          ansible_ssh_private_key_file: /path/to/key
    ```

!!! tip
    If your SSH private key is at a standard location on your Ansible Control
    Node (e.g. `~/.ssh/id_rsa`) you may omit `ansible_ssh_private_key_file`.

### Configure Cloud Credentials

This command will open MiCADO's cloud credential file in your preferred editor.
Most of our clouds are supported with username/password or key authentication.
```bash
micado config cloud
```

:   Provide credentials for one or more clouds, as required. Unused
    clouds can be left blank.

    ```yaml
    resource:
    - type: ec2
      auth_data:
        accesskey: ABC123DEF
        secretkey: 456XYZ789

    - type: cloudsigma
      auth_data:
        email: 
        password:

    - type: cloudbroker
      auth_data:
        email: user@example.com
        password: s3cur3_p4ssw0rd
    ```

The following clouds may require some additional explanation:

- OpenStack
- Google Cloud
- Microsoft Azure
- Oracle Cloud Infrastructure

???+ warning
    This file will be stored as a Kubernetes secret on the MiCADO Control Plane.
    To keep it secure on your Ansible Control Node, we recommend you encrypt it
    with [Ansible-Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).<br><br>
    We support only a single vault password. Remember to pass `‑‑vault` to `micado deploy`.

### Configure Proxy and TLS

MiCADO proxies a dashboard for inspecting the cluster and the submitter
endpoint for running application. This command configures basic auth
and the x509 keypair used for TLS encryption of the dashboard and submitter.

```bash
micado config web
```

:   See below for examples of this file. The username // password login
    for the MiCADO Dashboard will, in both cases, be `admin` // `admin_Pass`

    === "Self Signed"

        ```yaml
        tls:
          provision_method: self-signed
        authentication:
          username: admin
          email: user@example.com
          password: admin_Pass                 
        ```
        ??? info
            The self-signed option generates a new keypair with the specified
            hostname as the subject/CN.<br>This is `micado-master` by default,
            but can be configured with `micado config settings`.

            Two Subject Alternative Name (SAN) entries are also added:

            - DNS: *specified hostname*
            - IP: *specified IP*


    === "User Supplied"

        ```yaml
        tls:
          provision_method: user-supplied
          cert: |
            -----BEGIN CERTIFICATE-----
            MIID0DCCArigAwIBAgIBATANBgkqhkiG9w0BAQUFADB/MQswCQYDVQQGEwJGUjET
            ...
          key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEowIBAAKCAQEAvpnaPKLIKdvx98KW68lz8pGaRRcYersNGqPjpifMVjjE8LuC
            ...
        authentication:
          username: admin
          email: user@example.com
          password: admin_Pass
        ```

??? warning
    This file will be stored as a Kubernetes secret on the MiCADO Control Plane.
    To keep it secure on your Ansible Control Node, we recommend you encrypt it
    with [Ansible-Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).<br><br>
    We support only a single vault password. Remember to pass `‑‑vault` to `micado deploy`.

### Configure Container Registries

MiCADO uses **containerd** as the container runtime. The container
registries are configured via the the K3s `registries.yaml` file,
which is very well documented at
[this link](https://docs.k3s.io/installation/private-registry). The
following command will open that file for editing.

```bash
micado config registry
```

:   This simple example provides username and password for the
    Docker private registry, but much more is possible.

    ```yaml
    configs:
      registry-1.docker.io:
        auth:
          username: USERNAME
          password: PASSWORD
    ```

??? warning
    This file will be stored as a Kubernetes secret on the MiCADO Control Plane.
    To keep it secure on your Ansible Control Node, we recommend you encrypt it
    with [Ansible-Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).<br><br>
    We support only a single vault password. Remember to pass `‑‑vault` to `micado deploy`.

### Configure Additional Settings

Several additional options can be configured using the below command.

```bash
micado config settings
```

They will be detailed here.
