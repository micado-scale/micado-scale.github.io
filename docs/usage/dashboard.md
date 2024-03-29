# MiCADO Dashboard

MiCADO has a simple dashboard that collects several web-based user interfaces into
a single view. To access the Dashboard, visit `https://[IP]:[PORT]`, where

* [IP] is the ip address of the MiCADO Control Plane

* [PORT] is the [`web_listening_port`](/install/cli-install/#web-listening-port), which defaults to 443, but can be configured with `micado config settings`.

## Login

Login details are [configured during install](/install/cli-install/#configure-proxy-and-tls). If you opted for a self-signed certificate, you may have to bypass warnings generated by your browser. You should then be presented
with a login screen where you can enter the username and password you configured.

## Interfaces

The following interfaces are currently exposed:

### Kubernetes Dashboard

:   A *read-only* instance of the Kubernetes WebUI providing a full overview of the infrastructure.
    Since MiCADO provides its own authentication, you can **SKIP** the authentication pop-up to gain
    access to the dashboard.

    By default, your application will be scheduled in the `default` Kubernetes namespace.
    The Kubernetes Dashboard is well-documented in the official docs.

### Grafana

:   Graphically visualize the resources (nodes, containers) in time. After deploying your application, you can select the service whose metrics you want using the 'Service' drop down running above the graphs area.

### Prometheus

:   The monitoring subsystem. Recommended for developers, experts.
