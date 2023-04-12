# CloudSigma

To instantiate MiCADO workers on CloudSigma, please use the template below.
MiCADO **requires** num_cpus, mem_size, vnc_password, libdrive_id,
public_key_id and firewall_policy to instantiate VM on *CloudSigma*.

Currently, only **Occopus** has support for CloudSigma, so
[Occopus must be enabled](/install/cli-install/#enable-occopus), 
and the interface must be set to Occopus as in the example below.

```yaml
YOUR-VIRTUAL-MACHINE:
  type: tosca.nodes.MiCADO.CloudSigma.Compute
    properties:
      num_cpus: ADD_NUM_CPUS_FREQ (e.g. 4096)
      mem_size: ADD_MEM_SIZE (e.g. 4294967296)
      vnc_password: ADD_YOUR_PW (e.g. secret)
      libdrive_id: ADD_YOUR_ID_HERE (eg. 87ce928e-e0bc-4cab-9502-514e523783e3)
      public_key_id: ADD_YOUR_ID_HERE (e.g. d7c0f1ee-40df-4029-8d95-ec35b34dae1e)
      nics:
      - firewall_policy: ADD_YOUR_FIREWALL_POLICY_ID_HERE (e.g. fd97e326-83c8-44d8-90f7-0a19110f3c9d)
        ip_v4_conf:
          conf: dhcp

    interfaces:
      Occopus:
        create:
          inputs:
            endpoint: ADD_YOUR_ENDPOINT (e.g for cloudsigma https://zrh.cloudsigma.com/api/2.0 )
```

Under the **properties** section of a CloudSigma virtual machine definition
these inputs are available.:

* **num_cpus** is the speed of CPU (e.g. 4096) in terms of MHz of your VM
  to be instantiated. The CPU frequency required to be between 250 and 100000
* **mem_size** is the amount of RAM (e.g. 4294967296) in terms of bytes to be
  allocated for your VM. The memory required to be between 268435456 and
  137438953472
* **vnc_password** set the password for your VNC session (e.g. secret).
* **libdrive_id** is the image id (e.g. 87ce928e-e0bc-4cab-9502-514e523783e3)
  on your CloudSigma cloud. Select an image containing a base os installation
  with cloud-init support!
* **public_key_id** specifies the keypairs
  (e.g. d7c0f1ee-40df-4029-8d95-ec35b34dae1e) to be assigned to your VM.
* **nics[.firewall_policy && .ip_v4_conf.conf]**  specifies network policies
  (you can define multiple security groups in the form of a list for your VM).

## Interfaces

Under the **interfaces** section of a CloudSigma virtual machine definition,
remember to pass the endpoint of the CloudSigma region you wish to provision in.
