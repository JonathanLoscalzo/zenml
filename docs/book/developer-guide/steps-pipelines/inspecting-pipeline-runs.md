---
description: How to inspect a finished pipeline run
---

# Inspecting Pipeline Runs

Once a pipeline run has completed, we can access it using the
[ZenML Repository](../stacks-profiles-repositories/repository.md), about which you will find
more details in a later section.

Each pipeline can have multiple runs associated with it, and for each run there
might be several outputs for each step. Thus, to inspect a specific output, we
first need to access the respective pipeline, then fetch the respective run, 
and then choose the step output of that specific run.

The overall hierarchy looks like this:

```shell
repository -> pipelines -> runs -> steps -> outputs

# where -> implies a 1-many relationship.
```

Let us investigate how to traverse this hierarchy level by level:

### Repository

The highest level `Repository` object is where to start from.

```python
from zenml.repository import Repository


repo = Repository()
```

### Pipelines

The repository contains a collection of all created pipelines with at least one
run sorted by the time of their first run from oldest to newest.

You can either access this collection via the `get_pipelines()` method or query
a specific pipeline by name using `get_pipeline(pipeline_name=...)`:

```python
# get all pipelines from all stacks
pipelines = repo.get_pipelines()  

# now you can get pipelines by index
pipeline_with_latest_initial_run_time = pipelines[-1]

# or get one pipeline by name
pipeline_x = repo.get_pipeline(pipeline_name=...)
```

{% hint style="info" %}
Be careful when accessing pipelines by index. Even if you just ran a pipeline 
it might not be at index `-1`, due to the fact that the pipelines are sorted 
by time of *first* run. Instead, it is recommended to access the pipeline by its
name using `get_pipeline(pipeline_name=...)`.
{% endhint %}

### Runs

Each pipeline can be executed many times. You can get a list of all runs using
the `runs` attribute of a pipeline. Or, you can query a specific run by run
name using the `get_run(run_name=...)` method:

```python
# get all runs of a pipeline chronologically ordered
runs = pipeline_x.runs 

# get the last run by index, runs are ordered by execution time in ascending order
last_run = runs[-1]

# or get a specific run by name
run = pipeline_x.get_run(run_name=...)
```

{% hint style="warning" %}
Calling `pipeline.runs` can currently be very slow when using remote metadata
stores as all run data need to be transferred from the cloud to the local
machine. 
{% endhint %}

### Steps

Within a given pipeline run you can now further zoom in on individual steps
using the `steps` attribute or by querying a specific step using the
`get_step(name=...)` method.

```python
# get all steps of a pipeline for a given run
steps = run.steps

# get the step that was executed first
first_step = steps[0]

# or get a specific step by name
step = run.get_step(name=...)
```

{% hint style="info" %}
The steps are ordered by time of execution. Depending on the 
[orchestrator](../../mlops-stacks/orchestrators/orchestrators.md), steps can be run in 
parallel. Thus, accessing steps by index can be unreliable across different
runs and it is recommended to access steps by their name using
`get_step(name=...)` instead.
{% endhint %}

### Outputs

Finally, this is how you can inspect the output of a step:
- If there only is a single output, use the `output` attribute
- If there are multiple outputs, use the `outputs` attribute, which is a
dictionary that can be indexed using the name of an output:

```python
# The outputs of a step
# if there are multiple outputs they are accessible by name
output = step.outputs["output_name"]

# if there is only one output, use the `.output` property instead 
output = step.output 

# read the value into memory
output.read()  
```

{% hint style="info" %}
The names of the outputs can be found in the `Output` typing of your steps:

```python
from zenml.steps import step, Output


@step
def some_step() -> Output(output_name=int):
    ...
```
{% endhint %}

## Code Example

Putting it all together, this is how we can access the output of the last step
of our example pipeline from the previous sections:

```python
from zenml.repository import Repository


repo = Repository()
pipeline = repo.get_pipeline(pipeline_name="first_pipeline")
last_run = pipeline.runs[-1]
last_step = last_run.steps[-1]
model = last_step.output.read()
```