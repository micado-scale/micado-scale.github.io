# Install with Ansible

If you are familiar with Ansible and Ansible Playbooks, you are welcome to
configure and run the Playbooks without the CLI. The steps below provide
a rough outline to the required configurations.

!!! warning
    This page is currently under development, and sections
    may be outdated or incorrect.

## Installation

Perform the following steps either on your local machine or on MiCADO Control Plane depending on the installation method.

### Step 1: Download the ansible playbook

```bash
curl --output ansible-micado.tar.gz -L https://github.com/micado-scale/ansible-micado/releases/download/v0.12.0/micado-v0.12.0.tar.gz
tar -zxvf ansible-micado.tar.gz
```

### Step 2: Specify cloud credential for instantiating MiCADO workers.

MiCADO Control Plane will use the credentials against the cloud API to start/stop VM
instances (MiCADO workers) to host the application and to realize scaling.
Credentials here should belong to the same cloud as where MiCADO Control Plane
is running. We recommend making a copy of our predefined template and edit it.
MiCADO expects the credential in a file, called *credentials-cloud-api.yml*
before deployment. Please, do not modify the structure of the template!

```bash
cd playbook/project/credentials
cp sample-credentials-cloud-api.yml credentials-cloud-api.yml
edit credentials-cloud-api.yml
```

Edit **credentials-cloud-api.yml** to add cloud credentials. You will find
predefined sections in the template for each cloud interface type MiCADO
supports. It is recommended to fill only the section belonging to your
target cloud.

**NOTE** If you are using Google Cloud, you must replace or fill the
*credentials-gce.json* with your downloaded service account key file.

It is possible to modify cloud credentials after MiCADO has been deployed,
see the section titled **Update Cloud Credentials** further down this page

:   #### Optional: Added security

    Credentials are stored in Kubernetes Secrets on the MiCADO Control Plane. If
    you wish to keep the credential data in an secure format on the Ansible
    Remote as well, you can use the [Ansible Vault](https://docs.ansible.com/ansible/2.4/vault.html)
    mechanism to to achieve this. Simply create the above file using Vault with the
    following command

    ```
    ansible-vault create credentials-cloud-api.yml
    ```

    This will launch the editor defined in the `$EDITOR` environment variable to make changes to
    the file. If you wish to make any changes to the previously encrypted file, you can use the command
    
    ```
    ansible-vault edit credentials-cloud-api.yml
    ```

    Be sure to see the note about deploying a playbook with vault encrypted files
    in **Step 7**.

### Step 3a: Specify security settings and credentials to access MiCADO

MiCADO Control Plane will use these security-related settings and credentials to authenticate its users for accessing the REST API and Dashboard.

```
cp sample-credentials-micado.yml credentials-micado.yml
edit credentials-micado.yml
```

Specify the provisioning method for the x509 keypair used for TLS encryption of the management interface in the `tls` subtree:

:   - The **self-signed** option generates a new keypair with the specified
    hostname as the subject / CN ('micado-master' by default, but configurable in
    **micado-master.yml**).
    
        Two Subject Alternative Name (SAN) entries are also
        added by the configuration file at
        `roles/micado_master/start/templates/zorp/san.cnf`:
        
        - DNS: *specified hostname*
        - IP: *specified IP*

        The generated certificate file is located at:
        `/var/lib/micado/zorp/config/ssl.pem`


:   - The **user-supplied** option lets the user add the keypair as plain multiline strings (in unencrypted format) in the ansible_user_data.yml file under the 'cert' and 'key' subkeys respectively.

        Specify the default username and password for the administrative user in the `authentication` subtree.

**Optionally you may use the Ansible Vault mechanism as described in Step 2 to protect the confidentiality and integrity of this file as well.**


### Step 3b: (Optional) Specify credentials to use private Docker registries

Set the Docker login credentials of your private Docker registry in which your private containers are stored. We recommend making a copy of our predefined template and edit it. MiCADO expects the docker registry credentials in a file, called credentials-docker-registry.yml. Please, do not modify the structure of the template!

```
cp sample-credentials-registries.yml credentials-registries.yml
edit credentials-registries.yml
```

Edit the file according to the [K3s documentation](https://docs.k3s.io/installation/private-registry)

**Optionally you may use the Ansible Vault mechanism as described in Step 2 to protect the confidentiality and integrity of this file as well.**


### Step 4: Launch an empty cloud VM instance for MiCADO Control Plane

This new VM will host the MiCADO core services.

**a)** Default port number for MiCADO service is `443`. Optionally, you can modify the port number stored by the variable called `web_listening_port` defined in the ansible config file called `project/host_vars/micado.yml`.

**b)** Configure a cloud firewall settings which opens the following ports on the MiCADO Control Plane virtual machine:

|Protocol| Port(s)     |  Service           |
|--------|-------------|--------------------|
| TCP    |  443*       | web listening port |
| TCP    |  22         | SSH                |
| TCP    |  6443       | kube-apiserver     |
| TCP    |  10250      | metrics            |
| UDP    |  51820      | wireguard (Pod-Pod)|

   **NOTE:** `web listening port` should match with the actual value specified in Step 4a.


**c)** Finally, launch the virtual machine with the proper settings (capacity, ssh keys, firewall): use any of aws, ec2, nova, etc command-line tools or web interface of your target cloud to launch a new VM. We recommend a VM with 2 cores, 4GB RAM, 20GB disk. Make sure you can ssh to it (password-free i.e. ssh public key is deployed) and your user is able to sudo (to install MiCADO as root). Store its IP address which will be referred as `IP` in the following steps.

### Step 5: Customize the inventory file for the MiCADO Control Plane

We recommend making a copy of our predefined template and edit it. Use the template inventory file, called sample-hosts.yml for customisation.

```
cd playbook/inventory
cp sample-hosts.yml hosts.yml
edit hosts.yml
```

Edit the `hosts.yml` file to set the variables. The following parameters under the key **micado-target** can be updated:

* **ansible_host**: specifies the publicly reachable ip address of the target machine where you intend to build/deploy a MiCADO Control Plane or build a MiCADO Worker. Set the public or floating ``IP`` of the master regardless the deployment method is remote or local. The ip specified here is used by the Dashboard for webpage redirection as well
* **ansible_connection**: specifies how the target host can be reached. Use "ssh" for remote or "local" for local installation. In case of remote installation, make sure you can authenticate yourself against MiCADO Control Plane. We recommend to deploy your public ssh key on MiCADO Control Plane before starting the deployment
* **ansible_user**: specifies the name of your sudoer account, defaults to "ubuntu"
* **ansible_become**: specifies if account change is needed to become root, defaults to "True"
* **ansible_become_method**: specifies which command to use to become superuser, defaults to "sudo"
* **ansible_python_interpreter**: specifies the interpreter to be used for ansible on the target host, defaults to "/usr/bin/python3"

Please, revise all the parameters, however in most cases the default values are correct.


### Step 6: Customize the deployment

A few parameters in *project/host_vars/micado.yml* can be fine tuned before deployment. They are as follows:

- **enable_optimizer**: Setting this parameter to True enables the deployment of the Optimizer module, to perform more advanced scaling. Default is True.

- **disable_worker_updates**: Setting this parameter to False enables periodic software updates of the worker nodes. Default is True.

- **grafana_admin_pwd**: The string defined here will be the password for Grafana administrator.

- **web_listening_port**: Port number of the dasboard on MiCADO Control Plane. Default is 443.

- **web_session_timeout**: Timeout value in seconds for the Dashboard. Default is 600.

- **enable_occopus**: Install and enable Occopus for cloud orchestration. Default is True.

- **enable_terraform**: Install and enable Terraform for cloud orchestration. Default is False.

*Note. MiCADO supports running both Occopus & Terraform on the same Master, if desired*

### Step 7: Start the installation of MiCADO Control Plane

Run the following command to build and initalise a MiCADO Control Plane node on the empty VM you launched in Step 4 and pointed to in *hosts.yml* Step 5.

```
cd playbook/
ansible-playbook -i inventory/hosts.yml project/micado.yml
```

If you have used Vault to encrypt your credentials, you have to add the path to your vault credentials to the command line as described in the [Ansible Vault documentation](https://docs.ansible.com/ansible/2.4/vault.html#providing-vault-passwords) or provide it via command line using the command

```
cd playbook/
ansible-playbook -i inventory/hosts.yml project/micado.yml --ask-vault-pass
```

:   #### Optional: Build & Start Roles

    Optionally, you can split the deployment of your MiCADO Control Plane in two. The `build` tags prepare the node will all the necessary dependencies, libraries and images necessary for operation. The `start` tags intialise the cluster and all the MiCADO core components.

    You can clone the drive of a **"built"** MiCADO Control Plane (or otherwise make an image from it) to be reused again and again. This will greatly speed up the deployment of future instances of MiCADO.

    Running the following command will ``build`` a MiCADO Control Plane node on an empty Ubuntu VM.

    ```
    ansible-playbook -i inventory/hosts.yml project/micado.yml --tags build
    ```
    You can then run the following command to ``start`` any **"built"** MiCADO Control Plane node which will initialise and launch the core components for operation.

    ```
    ansible-playbook -i inventory/hosts.yml project/micado.yml --tags start
    ```


:   #### Advanced: Cloud specific fixes


    Certain cloud service providers may provide Virtual Machine images that are
    incompatible with the normal MiCADO installation. Where possible, we have included
    automated fixes for these, which can be applied using the `--tags` syntax of Ansible.
    See below for details:

    **CloudSigma**

    At the time of writing, the CloudSigma Ubuntu 20.04 virtual machine disk images
    are improperly configured, and SSL errors may appear during installation of MiCADO. A special
    task has been added to MiCADO to automate the fix when installing on CloudSigma instances.

    Simply use the following command instead of the command provided above. Notice the added tags

    ```
    ansible-playbook -i inventory/hosts.yml project/micado.yml --tags all,cloudsigma
    ```
