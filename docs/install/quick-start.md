# Quick-Start

This four step guide aims to get you started with MiCADO. For more detail on this installation
method, please see [CLI Install](cli-install.md).

First, provision a virtual machine in the cloud according to the [requirements](requirements.md#micado-control-plane).

Run the following commands from any device or instance that has SSH access to the newly provisioned instance. This can be a local device, or remote instance.


:   **1.** Install `micado-client`.

    ```bash
    pip install micado-client
    ```
<br>

:   **2.** Initialise a new directory for preparing the configuration. 
    <br>*The below example creates a new directory called* `micado_conf_dir`

    ```bash
    micado init micado_conf_dir
    cd micado_conf_dir

    ```
<br>

:   **3.** Configure your deployment. It is mandatory to configure `hosts`.

    === "hosts"
        ```bash
        micado config hosts
        ```

    === "cloud"
        ```bash
        micado config cloud 
        ```

    === "web"
        ```bash
        micado config web
        ```

    === "registry"
        ```bash
        micado config registry
        ```

    === "settings"
        ```bash
        micado config settings
        ```

:   !!! example
        <big>The command above will open the specified configuration in your preferred editor.</big>
        <br>Sample snippets of each config file are shown below.

    === "hosts"
        **Mandatory**. Configure IP and username for SSH access to the control plane node.<br>
        _If your SSH private key is **not** at a default path (e.g. .ssh/id_rsa) also specify path here_

        ```yaml
        all:
          hosts:
            micado:
              ansible_host: 123.456.78.90
              ansible_connection: ssh
              ansible_user: ubuntu
              ansible_ssh_private_key_file: /path/to/private/key
        ```

    === "cloud"
        Configure cloud credentials for desired clouds.<br>
        _Please consider using **ansible-vault** to encrypt this file_

        ```yaml
        resource:
        - type: ec2
          auth_data:
            accesskey: ABC123DEF
            secretkey: 456XYZ789
        ```

    === "web"
        Configure login and TLS for the MiCADO Dashboard and Submitter.<br>
        _Please consider using **ansible-vault** to encrypt this file_

        ```yaml
        tls:
          provision_method: self-signed
        authentication:
          username: admin
          email: user@example.com
          password: s3cur3p4ssw0rd
        ```

    === "registry"
        Configure private registry mirror/auth details as a
        [K3s registries file](https://docs.k3s.io/installation/private-registry#without-tls).

        ```yaml
        configs:
          registry-1.docker.io:
            auth:
              username: USERNAME
              password: PASSWORD
        ```

    === "settings"
        Configure various advanced settings. See the relevant section for details.

        ```yaml
        # enable specific components
        # -------------------------------------------------------
        enable_optimizer: False
        enable_occopus: False
        enable_terraform: True

        # enable multicloud support
        # -------------------------------------------------------
        enable_multicloud: False
        ```

: **4.** Deploy the MiCADO control plane.

    ```bash
    micado deploy
    ```

That's it! Have a look at our [demos](/demos/) to start deploying applications.