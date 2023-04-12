# Secrets policy

There is a way to define application-level secrets in the MiCADO application
description. These secrets are managed by Security Policy Manager and stored
and distributed as a single secret called **micado.appsecret** by Kubernetes.

See an example below for creating a secret using policies, and assigning
it to an environment variable (ENV_SALT in the example) in a container:


```yaml
topology_template:
  node_templates:
    my-app-container:
      type: tosca.nodes.MiCADO.Container.Application.Docker
      properties:
        ...
        env:
        - name: ENV_SALT
          valueFrom:
            secretKeyRef:
              name: micado.appsecret
              key: salt_value

  policies:
  - secret:
      type: tosca.policies.Security.MiCADO.Secret.KubernetesSecretDistribution
      properties:
        text_secrets:
          salt_value: "123456qwerty"
```