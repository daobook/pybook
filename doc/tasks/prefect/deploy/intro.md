# Deploy overview

Deployments allow you to run flows on a [schedule](https://docs.prefect.io/v3/automate/add-schedules/) and trigger runs based on [events](https://docs.prefect.io/v3/automate/events/automations-triggers/).

Deployments are server-side representations of flows.
They store the crucial metadata for remote orchestration including when, where, and how a workflow should run.

In addition to manually triggering and managing flow runs, deploying a flow exposes an API and UI that allow you to:

- trigger new runs, [cancel active runs](https://docs.prefect.io/v3/develop/write-flows/#cancel-a-flow-run), pause scheduled runs, customize parameters, and more
- remotely configure [schedules](https://docs.prefect.io/v3/automate/add-schedules) and [automation rules](https://docs.prefect.io/v3/automate/events/automations-triggers)
- dynamically provision infrastructure with [work pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools) - optionally with templated guardrails for other users

## Create a deployment 

### Create a deployment with `serve`

This is the simplest way to get started with deployments:

```python
from prefect import flow


@flow
def my_flow():
    print("Hello, Prefect!")


if __name__ == "__main__":
    my_flow.serve(name="my-first-deployment", cron="* * * * *")
```

The `serve` method creates a deployment from your flow and immediately begins listening for scheduled runs to execute. 
Providing `cron="* * * * *"` to `.serve` associates a schedule with your flow so it will run every minute of every day.

For more configuration, you can create a deployment that uses a work pool.
Reasons to create a work-pool based deployment include:

- Wanting to run your flow on dynamically provisioned infrastructure
- Needing more control over the execution environment on a per-flow run basis
- Creating an infrastructure template to use across deployments

Work pools are popular with data platform teams because they allow you to manage infrastructure configuration across an organization. 

### Create a work pool-based deployment

Prefect offers two options for creating work pool-based deployments: a Python script or a YAML file.

### Create a deployment with `deploy`

To define a deployment with a Python script, use the `flow.deploy` method.

Here's an example of a deployment that uses a work pool and bakes the code into a Docker image.

```python
from prefect import flow


@flow
def my_flow():
    print("Hello, Prefect!")

if __name__ == "__main__":
    my_flow.deploy(
        name="my-second-deployment",
        work_pool_name="my-work-pool",
        image="my-image",
        push=False,
        cron="* * * * *",
    )
```

To learn more about the `deploy` method, see [Deploy flows with Python](https://docs.prefect.io/v3/deploy/infrastructure-concepts/deploy-via-python).

### Create a deployment with a YAML file

If you'd rather take a declarative approach to defining a deployment through a YAML file, use a [`prefect.yaml` file](https://docs.prefect.io/v3/deploy/infrastructure-concepts/prefect-yaml).

Prefect provides an interactive CLI that walks you through creating a `prefect.yaml` file:

```bash
prefect deploy
```

The result is a `prefect.yaml` file for deployment creation. 
The file contains `build`, `push`, and `pull` steps for building a Docker image, pushing code to a Docker registry, and pulling code at runtime.
Learn more about creating deployments with a YAML file in [Define deployments with YAML](https://docs.prefect.io/v3/deploy/infrastructure-concepts/prefect-yaml).

Prefect also provides [CI/CD options](https://docs.prefect.io/v3/deploy/infrastructure-concepts/deploy-ci-cd) for automatically creating YAML-based deployments. 

### Work pools

[Work pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools/) allow you to switch between different types of infrastructure and to create a template for deployments. 
Data platform teams find work pools especially useful for managing infrastructure configuration across teams of data professionals.

Common work pool types include [Docker](https://docs.prefect.io/v3/deploy/infrastructure-examples/docker), [Kubernetes](https://docs.prefect.io/v3/deploy/infrastructure-examples/kubernetes), and serverless options such as [AWS ECS](/integrations/prefect-aws/ecs_guide#ecs-worker-guide), [Azure ACI](/integrations/prefect-azure/aci_worker), [GCP Vertex AI](/integrations/prefect-gcp/index#run-flows-on-google-cloud-run-or-vertex-ai), or [GCP Google Cloud Run](/integrations/prefect-gcp/gcp-worker-guide). 

### Work pool-based deployment requirements

Deployments created through the Python SDK that use a work pool require a `name`. 
This value becomes the deployment name.
A `work_pool_name` is also required.

Your flow code location can be specified in a few ways:

1. Bake it into your Docker image (for work-pools that use Docker images). 
    As shown in the example above,Prefect facilitates this as the default method for deployments created with the Python SDK. 
    This method requires that you specify the `image` argument in the `deploy` method.
2. Call `from_source` on a flow and specify one of the following:  
    1. the git-based cloud provider location (for example, GitHub) 
    2. the cloud provider storage location (for example, AWS S3)
    3. the local path (an option for Process work pools)

See the [Retrieve code from storage docs](https://docs.prefect.io/v3/deploy/infrastructure-concepts/store-flow-code) for more information about flow code storage.

## Run a deployment

You can set a deployment to run manually, on a [schedule](https://docs.prefect.io/v3/automate/add-schedules#schedule-flow-runs), or [in response to an event](https://docs.prefect.io/v3/automate/events/automations-triggers).

The deployment inherits the infrastructure configuration from the work pool, and can be overridden at deployment creation time or at runtime. 

### Work pools that require a worker

To run a deployment with a hybrid work pool type, such as Docker or Kubernetes, you must start a [worker](https://docs.prefect.io/v3/deploy/infrastructure-concepts/workers/).

A [Prefect worker](https://docs.prefect.io/v3/deploy/infrastructure-concepts/workers) is a client-side process that checks for scheduled flow runs in the work pool that it matches.
When a scheduled run is found, the worker kicks off a flow run on the specified infrastructure and monitors the flow run until completion.

### Work pools that don't require a worker

Prefect Cloud offers [push work pools](https://docs.prefect.io/v3/deploy/infrastructure-examples/serverless#automatically-create-a-new-push-work-pool-and-provision-infrastructure) that run flows on Cloud provider serverless infrastructure without a worker and that can be set up quickly. 

Prefect Cloud also provides the option to run work flows on Prefect's infrastructure through a [Prefect Managed work pool](https://docs.prefect.io/v3/deploy/infrastructure-examples/managed).

These work pool types do not require a worker to run flows. 
However, they do require sharing a bit more information with Prefect, which can be a challenge depending upon the security posture of your organization.

## Static vs. dynamic infrastructure

You can deploy your flows on long-lived static infrastructure or on dynamic infrastructure that is able to scale horizontally.
The best choice depends on your use case.

### Static infrastructure

When you have several flows running regularly, [the `serve` method](https://docs.prefect.io/v3/develop/write-flows/#serving-a-flow)
of the `Flow` object or [the `serve` utility](https://docs.prefect.io/v3/develop/write-flows/#serving-multiple-flows-at-once)
is a great option for managing multiple flows simultaneously.

Once you have authored your flow and decided on its deployment settings, run this long-running
process in a location of your choosing.
The process stays in communication with the Prefect API, monitoring for work and submitting each run
within an individual subprocess.
Because runs are submitted to subprocesses, any external infrastructure configuration
must be set up beforehand and kept associated with this process.

Benefits to this approach include:

- Users are in complete control of their infrastructure, and anywhere the "serve" Python process can
run is a suitable deployment environment.
- It is simple to reason about.
- Creating deployments requires a minimal set of decisions.
- Iteration speed is fast.

### Dynamic infrastructure

Consider running flows on dynamically provisioned infrastructure with work pools when you have any of the following:

- Flows that require expensive infrastructure due to the long-running process.
- Flows with heterogeneous infrastructure needs across runs.
- Large volumes of deployments.
- An internal organizational structure in which deployment authors or runners are not members of
the team that manages the infrastructure.

[Work pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools/) allow Prefect to exercise greater control
of the infrastructure on which flows run.
Options for [serverless work pools](https://docs.prefect.io/v3/deploy/infrastructure-examples/serverless/) allow you to
scale to zero when workflows aren't running.
Prefect even provides you with the ability to
[provision cloud infrastructure via a single CLI command](https://docs.prefect.io/v3/deploy/infrastructure-examples/serverless/#automatically-create-a-new-push-work-pool-and-provision-infrastructure),
if you use a Prefect Cloud push work pool option.

With work pools:

- You can configure and monitor infrastructure configuration within the Prefect UI.
- Infrastructure is ephemeral and dynamically provisioned.
- Prefect is more infrastructure-aware and collects more event data from your infrastructure by default.
- Highly decoupled setups are possible.

<Note>
**You don't have to commit to one approach**

You can mix and match approaches based on the needs of each flow. You can also change the
deployment approach for a particular flow as its needs evolve.
For example, you might use workers for your expensive machine learning pipelines,
but use the serve mechanics for smaller, more frequent file-processing pipelines.
</Note>

## Deployment schema

{/* pmd-metadata: notest */}
```python
class Deployment:
    """
    Structure of the schema defining a deployment
    """

    # required defining data
    name: str
    flow_id: UUID
    entrypoint: str
    path: Optional[str] = None

    # workflow scheduling and parametrization
    parameters: Optional[Dict[str, Any]] = None
    parameter_openapi_schema: Optional[Dict[str, Any]] = None
    schedules: list[Schedule] = None
    paused: bool = False
    trigger: Trigger = None

    # concurrency limiting
    concurrency_limit: Optional[int] = None
    concurrency_options: Optional[
        ConcurrencyOptions(collision_strategy=Literal['ENQUEUE', 'CANCEL_NEW'])
    ] = None

    # metadata for bookkeeping
    version: Optional[str] = None
    description: Optional[str] = None
    tags: Optional[list] = None

    # worker-specific fields
    work_pool_name: Optional[str] = None
    work_queue_name: Optional[str] = None
    job_variables: Optional[Dict[str, Any]] = None
    pull_steps: Optional[Dict[str, Any]] = None
```

All methods for creating Prefect deployments are interfaces for populating this schema.

### Required defining data

Deployments require a `name` and a reference to an underlying `Flow`.

The deployment name is not required to be unique across all deployments, but is required to be unique
for a given flow ID. This means you will often see references to the deployment's unique identifying name
`{FLOW_NAME}/{DEPLOYMENT_NAME}`.

For example, to trigger a deployment run from the Prefect CLI:

```bash
prefect deployment run my-first-flow/my-first-deployment
```

or from the Python SDK with [`run_deployment`](https://prefect-python-sdk-docs.netlify.app/prefect/deployments/flow_runs/#prefect.deployments.flow_runs.run_deployment):

{/* pmd-metadata: notest */}
```python
from prefect.deployments import run_deployment


run_deployment(
    name="my-first-flow/my-first-deployment",
    parameters={"my_param": "42"},
    job_variables={"env": {"MY_ENV_VAR": "staging"}},
    timeout=0, # don't wait for the run to finish
)
```

The other two fields are:

- **`path`**: think of the path as the runtime working directory for the flow.
For example, if a deployment references a workflow defined within a Docker image, the `path` is the
absolute path to the parent directory where that workflow will run anytime the deployment is triggered.
This interpretation is more subtle in the case of flows defined in remote filesystems.
- **`entrypoint`**: the entrypoint of a deployment is a relative reference to a function decorated as a
flow that exists on some filesystem. It is always specified relative to the `path`.
Entrypoints use Python's standard path-to-object syntax
(for example, `path/to/file.py:function_name` or simply `path:object`).

The entrypoint must reference the same flow as the flow ID.

Prefect requires that deployments reference flows defined _within Python files_.
Flows defined within interactive REPLs or notebooks cannot currently be deployed as such.
They are still valid flows that will be monitored by the API and observable in the UI whenever they are run,
but Prefect cannot trigger them.

<Note>
**Deployments do not contain code definitions**

Deployment metadata references code that exists in potentially diverse locations within your environment.
This separation means that your flow code stays within your storage and execution
infrastructure.

This is key to the Prefect hybrid model: there's a boundary between your proprietary assets,
such as your flow code, and the Prefect backend (including [Prefect Cloud](https://docs.prefect.io/v3/manage/cloud/)).
</Note>

### Workflow scheduling and parametrization

One of the primary motivations for creating deployments of flows is to remotely _schedule_ and _trigger_ them.
Just as you can call flows as functions with different input values, deployments can be triggered or
scheduled with different values through parameters.

These are the fields to capture the required metadata for those actions:

- **`schedules`**: a list of [schedule objects](https://docs.prefect.io/v3/automate/add-schedules/).
Most of the convenient interfaces for creating deployments allow users to avoid creating this object themselves.
For example, when [updating a deployment schedule in the UI](https://docs.prefect.io/v3/automate/add-schedules/#creating-schedules-through-the-ui)
basic information such as a cron string or interval is all that's required.
- **`parameter_openapi_schema`**: an [OpenAPI compatible schema](https://swagger.io/specification/) that defines
the types and defaults for the flow's parameters.
This is used by the UI and the backend to expose options for creating manual runs as well as type validation.
- **`parameters`**: default values of flow parameters that this deployment will pass on each run.
These can be overwritten through a trigger or when manually creating a custom run.
- **`enforce_parameter_schema`**: a boolean flag that determines whether the API should validate the parameters
passed to a flow run against the schema defined by `parameter_openapi_schema`.

<Tip>
**Scheduling is asynchronous and decoupled**

Pausing a schedule, updating your deployment, and other actions reset your auto-scheduled runs.
</Tip>

### Concurrency limiting

Prefect supports managing concurrency at the deployment level to enable limiting how many runs of a
deployment can be active at once. To enable this behavior, deployments have the following fields:

- **`concurrency_limit`**: an integer that sets the maximum number of concurrent flow runs for the deployment.
- **`collision_strategy`**: configure the behavior for runs once the concurrency limit is reached.
Falls back to `ENQUEUE` if unset.
  - `ENQUEUE`: new runs transition to `AwaitingConcurrencySlot` and execute as slots become available.
  - `CANCEL_NEW`: new runs are canceled until a slot becomes available.

<CodeGroup>

```sh prefect deploy
prefect deploy ... --concurrency-limit 3 --collision-strategy ENQUEUE
```

```python flow.deploy()
from prefect.client.schemas.objects import (
    ConcurrencyLimitConfig, 
    ConcurrencyLimitStrategy
)

my_flow.deploy(..., concurrency_limit=3)

my_flow.deploy(
    ...,
    concurrency_limit=ConcurrencyLimitConfig(
        limit=3, collision_strategy=ConcurrencyLimitStrategy.CANCEL_NEW
    ),
)
```

```python flow.serve()
from prefect.client.schemas.objects import (
    ConcurrencyLimitConfig, 
    ConcurrencyLimitStrategy
)

my_flow.serve(..., global_limit=3)

my_flow.serve(
    ...,
    global_limit=ConcurrencyLimitConfig(
        limit=3, collision_strategy=ConcurrencyLimitStrategy.CANCEL_NEW
    ),
)
```

</CodeGroup>

### Metadata for bookkeeping

Important information for the versions, descriptions, and tags fields:

- **`version`**: versions are always set by the client and can be any arbitrary string.
We recommend tightly coupling this field on your deployments to your software development lifecycle.
For example if you leverage `git` to manage code changes, use either a tag or commit hash in this field.
If you don't set a value for the version, Prefect will compute a hash.
- **`description`**: provide reference material such as intended use and parameter documentation.
Markdown is accepted. The docstring of your flow function is the default value.
- **`tags`**: group related work together across a diverse set of objects.
Tags set on a deployment are inherited by that deployment's flow runs. Filter, customize views, and
searching by tag.

<Tip>
**Everything has a version**

Deployments have a version attached; and flows and tasks also have
versions set through their respective decorators. These versions are sent to the API
anytime the flow or task runs, allowing you to audit changes.
</Tip>

### Worker-specific fields

[Work pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools/) and [workers](https://docs.prefect.io/v3/deploy/infrastructure-concepts/workers/) are an advanced deployment pattern that
allow you to dynamically provision infrastructure for each flow run.
The work pool job template interface allows users to create and govern opinionated interfaces
to their workflow infrastructure.
To do this, a deployment using workers needs the following fields:

- **`work_pool_name`**: the name of the work pool this deployment is associated with.
Work pool types mirror infrastructure types, which means this field impacts the options available
for the other fields.
- **`work_queue_name`**: if you are using work queues to either manage priority or concurrency, you can
associate a deployment with a specific queue within a work pool using this field.
- **`job_variables`**: this field allows deployment authors to customize whatever infrastructure
options have been exposed on this work pool.
This field is often used for Docker image names, Kubernetes annotations and limits,
and environment variables.
- **`pull_steps`**: a JSON description of steps that retrieves flow code or
configuration, and prepares the runtime environment for workflow execution.

Pull steps allow users to highly decouple their workflow architecture.
For example, a common use of pull steps is to dynamically pull code from remote filesystems such as
GitHub with each run of their deployment.