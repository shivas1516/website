+++
title = "Using environment variables in pipelines"
description = "How to set and use environment variables in Kubeflow pipelines"
weight = 115
                    
+++
{{% alert title="Old Version" color="warning" %}}
This page is about __Kubeflow Pipelines V1__, please see the [V2 documentation](/docs/components/pipelines) for the latest information.

Note, while the V2 backend is able to run pipelines submitted by the V1 SDK, we strongly recommend [migrating to the V2 SDK](/docs/components/pipelines/user-guides/migration).
For reference, the final release of the V1 SDK was [`kfp==1.8.22`](https://pypi.org/project/kfp/1.8.22/), and its reference documentation is [available here](https://kubeflow-pipelines.readthedocs.io/en/1.8.22/).
{{% /alert %}}

This page describes how to pass environment variables to Kubeflow pipeline 
components.

## Before you start

Set up your environment: 

- [Install Kubeflow](/docs/started/)
- [Install the Kubeflow Pipelines SDK](/docs/components/pipelines/legacy-v1/sdk/install-sdk/)



## Using environment variables 

In this example, you pass an environment variable to a lightweight Python 
component, which writes the variable's value to the log.

[Learn more about lightweight Python components](/docs/components/pipelines/user-guides/components/lightweight-python-components/)

To build a component, define a stand-alone Python function and then call 
[kfp.components.func_to_container_op(func)](https://kubeflow-pipelines.readthedocs.io/en/stable/source/components.html#kfp.components.func_to_container_op) to convert the 
function to a component that can be used in a pipeline. The following function gets an 
environment variable and writes it to the log.

```python
def logg_env_function():
  import os
  import logging
  logging.basicConfig(level=logging.INFO)
  env_variable = os.getenv('example_env')
  logging.info('The environment variable is: {}'.format(env_variable))
```

Transform the function into a component using 
[kfp.components.func_to_container_op(func)](https://kubeflow-pipelines.readthedocs.io/en/stable/source/components.html#kfp.components.func_to_container_op).  
```python
image_name = 'tensorflow/tensorflow:1.11.0-py3'
logg_env_function_op = comp.func_to_container_op(logg_env_function,
                                                 base_image=image_name)
```

Add this component to a pipeline. Use [add_env_variable](https://kubeflow-pipelines.readthedocs.io/en/stable/source/dsl.html#kfp.dsl.ContainerOp.container) to pass an 
environment variable into the component. This code is the same no matter if your
using python lightweight components or a [container operation](https://kubeflow-pipelines.readthedocs.io/en/stable/source/dsl.html#kfp.dsl.ContainerOp). 


```python
import kfp.dsl as dsl
from kubernetes.client.models import V1EnvVar

@dsl.pipeline(
  name='Env example',
  description='A pipeline showing how to use environment variables'
)
def environment_pipeline():
  env_var = V1EnvVar(name='example_env', value='env_variable')
  #Returns a dsl.ContainerOp class instance. 
  container_op = logg_env_function_op().add_env_variable(env_var) 
```

To pass more environment variables into a component, add more instances of 
[add_env_variable()](https://kubeflow-pipelines.readthedocs.io/en/stable/source/dsl.html#kfp.dsl.ContainerOp.container). Use the following command to run this pipeline using the 
Kubeflow Pipelines SDK.

```python
#Specify pipeline argument values
arguments = {}

#Submit a pipeline run
kfp.Client().create_run_from_pipeline_func(environment_pipeline,
                                           arguments=arguments)
```
