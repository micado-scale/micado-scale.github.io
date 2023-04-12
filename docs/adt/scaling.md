# Scaling Policies


## Basic scaling

To utilise the autoscaling functionality of MiCADO, scaling policies can be defined on virtual machine and on the application level. Scaling policies can be listed under the **policies** section. Each **scalability** subsection must have the **type** set to the value of `tosca.policies.Scaling.MiCADO` and must be linked to a node defined under **node_template**. The link can be implemented by specifying the name of the node under the **targets** subsection. You can attach different policies to different containers or virtual machines, though a new policy should exist for each. The details of the scaling policy can be defined under the **properties** subsection. The structure of the **policies** section can be seen below.

```yaml
  topology_template:
    node_templates:
      YOUR-VIRTUAL-MACHINE:
        type: tosca.nodes.MiCADO.<CLOUD_API_TYPE>.Compute
        ...
      YOUR-OTHER-VIRTUAL-MACHINE:
        type: tosca.nodes.MiCADO.<CLOUD_API_TYPE>.Compute
        ...
      YOUR-KUBERNETES-APP:
        type: tosca.nodes.MiCADO.Container.Application.Docker
        ...
      YOUR-OTHER-KUBERNETES-APP:
        type: tosca.nodes.MiCADO.Container.Application.Docker
        ...

    policies:
    - scalability:
      type: tosca.policies.Scaling.MiCADO
      targets: [ YOUR-VIRTUAL-MACHINE ]
      properties:
        ...
    - scalability:
      type: tosca.policies.Scaling.MiCADO
      targets: [ YOUR-OTHER-VIRTUAL-MACHINE ]
      properties:
        ...
    - scalability:
      type: tosca.policies.Scaling.MiCADO
      targets: [ YOUR-KUBERNETES-APP ]
      properties:
        ...
    - scalability:
      type: tosca.policies.Scaling.MiCADO
      targets: [ YOUR-OTHER-KUBERNETES-APP ]
      properties:
        ...
```

The scaling policies are evaluated periodically. In every turn, the virtual machine level scaling policies are evaluated, followed by the evaluation of each scaling policies belonging to kubernetes-deployed applications.

### Properties

The **properties** subsection defines the scaling policy itself. For monitoring purposes, MiCADO integrates the Prometheus monitoring tool with two built-in exporters on each worker node: Node exporter (to collect data on nodes) and CAdvisor (to collect data on containers). Based on Prometheus, any monitored information can be extracted using the Prometheus query language and the returned value can be associated to a user-defined variable. Once variables are updated, scaling rule is evaluated. Scaling rule is specified by (a short) Python code. The code can refer to/use the variables. The structure of the scaling policy can be seen below.

```yaml
  - scalability:
      ...
      properties:
        sources:
          - 'myprometheus.exporter.ip.address:portnumber'
        constants:
          LOWER_THRESHOLD: 50
          UPPER_THRESHOLD: 90
          MYCONST: 'any string'
        queries:
          THELOAD: 'Prometheus query expression returning a number'
          MYLISTOFSTRING: ['Prometheus query returning a list of strings as tags','tagname as filter']
          MYEXPR: 'something refering to {{MYCONST}}'
        alerts:
          - alert: myalert
            expr: 'Prometheus expression for an event important for scaling'
            for: 1m
        min_instances: 1
        max_instances: 5
        scaling_rule: |
          if myalert:
            m_node_count=5
          if THELOAD>UPPER_THRESHOLD:
            m_node_count+=1
          if THELOAD<LOWER_THRESHOLD:
            m_node_count-=1
```

The subsections have the following roles:

* **sources** supports the dynamic attachment of an external exporter by specifying a list endpoints of exporters (see example above). Each item found under this subsection is configured under Prometheus to start collecting the information provided/exported by the exporters. Once done, the values of the parameters provided by the exporters become available. MiCADO supports Kubernetes service discovery to define such a source, simply pass the name of the app as defined in TOSCA and do not specify any port number
* **constants** subsection is used to predefined fixed parameters. Values associated to the parameters can be referred by the scaling rule as variable (see ``LOWER_THRESHOLD`` above) or in any other sections referred as Jinja2 variable (see ``MYEXPR`` above).
* **queries** contains the list of Prometheus query expressions to be executed and their variable name associated (see ``THELOAD`` or ``MYLISTOFSTRING`` above)
* **alerts** subsection enables the utilization of the alerting system of Prometheus. Each alert defined here is registered under Prometheus and fired alerts are represented with a variable of their name set to True during the evaluation of the scaling rule (see ``myalert`` above).
* **min_instances** keyword specifies the lowest number of instances valid for the node.
* **max_instances** keyword specifies the highest number of instances valid for the node.
* **scaling_rule** specifies Python code to be evaluated periodically to decide on the number of instances. The Python expression must be formalized with the following conditions:

  - Each constant defined under the ‘constants’ section can be referred; its value is the one defined by the user.
  - Each variable defined under the ‘queries’ section can be referred; its value is the result returned by Prometheus in response to the query string.
  - Each alert name defined under the ‘alerts’ section can be referred, its value is a logical True in case the alert is firing, False otherwise
  - Expression must follow the syntax of the Python language
  - Expression can be multiline
  - The following predefined variables can be referred; their values are defined and updated before the evaluation of the scaling rule

    - m_nodes: python list of nodes belonging to the kubernetes cluster
    - m_node_count: the target number of nodes
    - m_nodes_todrop: the ids or ip addresses of the nodes to be dropped in case of downscaling **NOTE MiCADO-Terraform supports private IPs on Azure or AWS EC2 only**
    - m_container_count: the target number of containers for the service the evaluation belongs to
    - m_time_since_node_count_changed: time in seconds elapsed since the number of nodes changed

  - In a scaling rule belonging to the virtual machine, the name of the variable to be updated is ``m_node_count``; as an effect the number stored in this variable will be set as target instance number for the virtual machines.
  - In a scaling rule belonging to the virtual machine, the name of the variable to be updated is ``m_nodes_todrop``;the variable must be filled with list of ids or ip addresses and as an effect the valid nodes will be dropped. The variable ``m_node_count`` should not be modified in case of node dropping, MiCADO will update it automatically.
  - In a scaling rule belonging to a kubernetes deployment, the name of the variable to be set is ``m_container_count``; as an effect the number stored in this variable will be set as target instance number for the kubernetes service.

For debugging purposes, the following support is provided:

* ``m_dryrun`` can be specified in the **constant** as list of components towards which the communication is disabled. It has the following syntax: m_dryrun: ["prometheus","occopus","k8s","optimizer"] Use this feature with caution!

* the standard output of the python code defined by the user under the scaling rule section is collected in a separate log file stored under the policy keeper log directory. It can also be used for debugging purposes.

For further examples, inspect the scaling policies of the demo examples detailed in the next section.

## Optimiser-based scaling

For implementing more advanced scaling policies, it is possible to utilize the built-in Optimiser in MiCADO. The role of the Optimiser is to support decision making in calculating the number of worker nodes (virtual machines) i.e. to scale the nodes to the optimal level. Optimiser is implemented using machine learning algorithm aiming to learn the relation between various metrics and the effect of scaling events. Based on this learning, the Optimiser is able to calculate and advise on the necessary number of virtual machines.

Current limitations
  - only web based applications are supported
  - only one of the node sets can be supported
  - no container scaling is supported

Optimiser can be utilised based on the following principles
  - User specifies a so-called target metric with its associated minimum and maximum thresholds. The target metric is a monitored Prometheus expression for which the value is tried to be kept between the two thresholds by the Optimiser with scaling advices.
  - User specifies several so-called input metrics which represent the state of the system correlating to the target variable
  - User specifies several initial settings (see later) for the Optimiser
  - User submits the application activating the Optimiser through the ADT
  - Optimiser starts with the 'training' phase during which the correlations are learned. During the training phase artificial load must be generated for the web application and scaling activities must be performed (including extreme values) in order to present all possible situations for the Optimiser. During the phase, Optimiser continuously monitors the input/target metrics and learns the correlations.
  - When correlations are learnt, Optimiser turns to 'production' phase during which advice can be requested from the Optimiser. During this phase, Optimiser returns advice on request, where the advice contains the number of virtual machines (nodes) to be scaled to. During the production phase, the Optimiser continues its learning activity to adapt to the new situations.

Activation of the Optimiser
  Optimiser must be enabled at deployment time. By default it is disabled. Once it is enabled and deployed, it can be driven through the scaling policy in subsections "constants" and "queries". Each parameter relating to the Optimiser must start with the "m_opt\_" string. In case no variable name with this prefix is found in any sections, Optimiser is not activated.

Initial settings for the Optimiser
  Parameters for initial settings are defined under the "constants" section and their name must start with the `m_opt_init_` prefix. These parameters are as follows:

  - **m_opt_init_knowledge_base** is a parameter which specifies the way how the knowledge base must be built under the Optimiser. When defined as "build_new", Optimiser empties its knowledge base and starts building a new knowledge i.e. starts learning the correlations. When using the "use_existing" value, the knowledge is kept and continued building further. Default is "use_existing".
  - **m_opt_init_training_samples_required** defines how many sample of the metrics must be collected by the Optimiser before start learning the correlations. Default is 300.
  - **m_opt_init_max_upscale_delta** specifies the maximum change in number of node for an upscaling advice. Default is 6.
  - **m_opt_init_max_downscale_delta** specifies the maximum change in number of node for a downscaling advice. Default is 6.
  - **m_opt_init_advice_freeze_interval** specifies how many seconds must elapse before the Optimiser advises a different number of node. Can be used to mitigate the frequency of scaling. Defaults to 0.

Definition of input metrics for the Optimizer
  Input metrics must be specified for the Optimiser under the "queries" subsection to perform the training i.e. learning the correlations. Each parameter must start with the "m_opt_input\_" prefix, e.g. m_opt_input_CPU. The following two pieces of variable must be specified for the web application:

  - **m_opt_input_AVG_RR** should specify the average request rate of the web server.
  - **m_opt_input_SUM_RR** should specify the summary of request rate of the web server.

Definition of the target metric for the Optimizer
  Target metric is a continuously monitored parameter that must be kept between thresholds. To specify it, together with the thresholds, "m_opt_target\_" prefix must be used. These three parameter must be defined under the "queries" sections. They are as follows:

  - **m_opt_target_query_MYTARGET** specifies the prometheus query for the target metric called MYTARGET.
  - **m_opt_target_minth_MYTARGET** specifies the value above which the target metric must be kept.
  - **m_opt_target_maxth_MYTARGET** specifies the value below which the target metric must be kept.

Requesting scaling advice from the Optimizer
  In order to receive a scaling advice from the Optimiser, the method **m_opt_advice()** must be invoked in the scaling_rule section of the node.

  **IMPORTANT! Minimum and maximum one node must contain this method invocation in its scaling_rule section for proper operation!**

  The **m_opt_advice()** method returns a python dictionary containing the following fields:

  - **valid** stores True/False value indicating whether the advise can be considered or not.
  - **phase** indicates whether the Optimiser is in "training" or "production" phase.
  - **vm_number** represents the advise for the target number of nodes to scale to.
  - **reliability** represents the goodness of the advice with a number between 0 and 100. The bigger the number is the better/more reliable the advice is.
  - **error_msg** contains the error occured in the Optimiser. Filled when valid is False.
