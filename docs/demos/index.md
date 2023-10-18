# Demos

There are a number of demo applications that come bundled with MiCADO.
After deploying MICADO you are welcome to try out the demos to get
a feel for what MiCADO can do.

Each demo works the same way - you must edit the ADT and provide
cloud configuration details to describe an instance that will
host the demo in the cloud. 

For the NGINX demo on EC2, you can open the ADT like so:

```bash
micado demo nginx ec2
```

To then run the demo, use the MiCADO CLI:

```bash
micado start nginx_ec2.yaml
```

Explore the Dashboard, and once you are done, shut down the
demo with the MiCADO CLI:

```bash
micado stop
```