# Deploy flows with Python

Prefect offers a flexible way to deploy flows to dynamic infrastructure using the Python SDK. 
This approach allows you to target specific work pools and utilize dynamically provisioned infrastructure.

Deploying flows to dynamic infrastructure offers a flexible and programmatic approach to managing your 
Prefect workflows. While `flow.serve` remains a valid and straightforward method for deploying flows to persistent 
infrastructure, there are scenarios where you might need to leverage dynamic, scalable resources. In these cases, 
`flow.deploy` provides a powerful alternative that allows you to target specific work pools and utilize dynamically 
provisioned infrastructure.

## When to consider `flow.deploy` over `flow.serve`

The `flow.serve` method is an excellent choice for many use cases, especially when you have readily available, persistent infrastructure. However, you might want to consider using `flow.deploy` in the following situations:

1. Cost optimization: Dynamic infrastructure can help reduce costs by scaling resources up or down based on demand.
2. Resource scarcity: If you have limited persistent infrastructure, dynamic provisioning can help manage resource allocation more efficiently.
3. Varying workloads: For workflows with inconsistent resource needs, dynamic infrastructure can adapt to changing requirements.
4. Cloud-native deployments: When working with cloud providers that offer serverless or auto-scaling options.

By using `flow.deploy`, you can take advantage of Prefect's work pool system, which provides a layer of abstraction between your flows and the underlying infrastructure. This approach offers greater flexibility and efficiency in running your workflows when dynamic infrastructure is needed.

Let's explore how to create a deployment using the Python SDK and leverage dynamic infrastructure through work pools.

## Prerequisites

Before deploying your flow using `flow.deploy`, ensure you have the following:

1. A running Prefect server or Prefect Cloud workspace: You can either run a Prefect server locally or use a Prefect Cloud workspace. 
To start a local server, run `prefect server start`. To use Prefect Cloud, sign up for an account at [app.prefect.cloud](https://app.prefect.cloud) 
and follow the [Connect to Prefect Cloud](https://docs.prefect.io/v3/manage/cloud/connect-to-cloud/) guide.

2. A Prefect flow: You should have a flow defined in your Python script. If you haven't created a flow yet, refer to the 
[Write Flows](https://docs.prefect.io/v3/develop/write-flows/) guide.

3. A work pool: You need a work pool to manage the infrastructure for running your flow. If you haven't created a work pool, you can do so through 
the Prefect UI or using the Prefect CLI. For more information, see the [Work Pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools/) guide. 

    For examples in this guide, we'll use a Docker work pool created by running:
    ```bash
    prefect work-pool create --type docker my-work-pool
    ```

4. Docker: Docker will be used to build and store the image containing your flow code. You can download and install
Docker from the [official Docker website](https://www.docker.com/get-started/). If you don't want to use Docker, you 
can see other options in the [Use remote code storage](#use-remote-code-storage) section.

5. (Optional) A Docker registry account: While not strictly necessary for local development, having an account with a Docker registry (such as 
Docker Hub) is recommended for storing and sharing your Docker images.


With these prerequisites in place, you're ready to deploy your flow using `flow.deploy`.


## Deploy a flow with `flow.deploy`

To deploy a flow using `flow.deploy` and Docker, follow these steps:

<Steps>
    <Step title="Write a flow">
        Ensure your flow is defined in a Python file. Let's use a simple example:
        ```python example.py
        from prefect import flow


        @flow(log_prints=True)
        def my_flow(name: str = "world"):
            print(f"Hello, {name}!")
        ```
    </Step>
    <Step title="Add deployment configuration">
        Add a call to `flow.deploy` to tell Prefect how to deploy your flow.
        ```python example.py
        from prefect import flow


        @flow(log_prints=True)
        def my_flow(name: str = "world"):
            print(f"Hello, {name}!")


        if __name__ == "__main__":
            my_flow.deploy(
                name="my-deployment",
                work_pool_name="my-work-pool",
                image="my-docker-image:dev",
                push=False
            )
        ```
    </Step>
    <Step title="Deploy!">
        Run your script to deploy your flow.
        ```bash
        python example.py
        ```

        Running this script will:

        1. Build a Docker image containing your flow code and dependencies.
        2. Create a deployment associated with the specified work pool and image.
    </Step>
</Steps>

Building a Docker image for our flow allows us to have a consistent environment for our flow to run in. Workers for our work pool will use the image to run our flow.
In this example, we set `push=False` to skip pushing the image to a registry. This is useful for local development, and you can push your image to a registry such as Docker Hub in a production environment.

<Note>
**Where's the Dockerfile?**

In the above example, we didn't specify a Dockerfile. By default, Prefect will generate a Dockerfile for us that copies the flow code into an image and installs 
any additional dependencies.

If you want to write and use your own Dockerfile, you can do so by passing a `dockerfile` parameter to `flow.deploy`.
</Note>

### Trigger a run

Now that we have our flow deployed, we can trigger a run via either the Prefect CLI or UI.

First, we need to start a worker to run our flow:

```bash
prefect worker start --pool my-work-pool
```

Then, we can trigger a run of our flow using the Prefect CLI:

```bash
prefect deployment run 'my-flow/my-deployment'
```

After a few seconds, you should see logs from your worker showing that the flow run has started and see the state update in the UI.

## Deploy with a schedule

To deploy a flow with a schedule, you can use one of the following options:

- `interval`

    Defines the interval at which the flow should run. Accepts an integer or float value representing the number of seconds between runs or a `datetime.timedelta` object.

    <Expandable title="Example">
    ```python interval.py
    from datetime import timedelta
    from prefect import flow


    @flow(log_prints=True)
    def my_flow(name: str = "world"):
        print(f"Hello, {name}!")


    if __name__ == "__main__":
        my_flow.deploy(
            name="my-deployment",
            work_pool_name="my-work-pool",
            image="my-docker-image:dev",
            push=False,
            # Run once a minute
            interval=timedelta(minutes=1)
        )
    ```
    </Expandable>

- `cron`

    Defines when a flow should run using a cron string.
    <Expandable title="Example">
    ```python cron.py
    from prefect import flow


    @flow(log_prints=True)
    def my_flow(name: str = "world"):
        print(f"Hello, {name}!")


    if __name__ == "__main__":
        my_flow.deploy(
            name="my-deployment",
            work_pool_name="my-work-pool",
            image="my-docker-image:dev",
            push=False,
            # Run once a day at midnight
            cron="0 0 * * *"
        )
    ```
    </Expandable>

- `rrule`

    Defines a complex schedule using an `rrule` string.

    <Expandable title="Example">
    ```python rrule.py
    from prefect import flow


    @flow(log_prints=True)
    def my_flow(name: str = "world"):
        print(f"Hello, {name}!")


    if __name__ == "__main__":
        my_flow.deploy(
            name="my-deployment",
            work_pool_name="my-work-pool",
            image="my-docker-image:dev",
            push=False,
            # Run every weekday at 9 AM
            rrule="FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;BYHOUR=9;BYMINUTE=0"
        )
    ```
    </Expandable>

- `schedules`

    Defines multiple schedules for a deployment. This option provides flexibility for:
        - Setting up various recurring schedules
        - Implementing complex scheduling logic
        - Applying timezone offsets to schedules

    <Expandable title="Example">
    ```python schedules.py
    from datetime import datetime, timedelta
    from prefect import flow
    from prefect.client.schemas.schedules import IntervalSchedule


    @flow(log_prints=True)
    def my_flow(name: str = "world"):
        print(f"Hello, {name}!")


    if __name__ == "__main__":
        my_flow.deploy(
            name="my-deployment",
            work_pool_name="my-work-pool",
            image="my-docker-image:dev",
            push=False,
            # Run every 10 minutes starting from January 1, 2023
            # at 00:00 Central Time
            schedules=[
                IntervalSchedule(
                    interval=timedelta(minutes=10),
                    anchor_date=datetime(2023, 1, 1, 0, 0),
                    timezone="America/Chicago"
                )
            ]
        )
    ```
    </Expandable>

    Learn more about schedules [here](https://docs.prefect.io/v3/automate/add-schedules).


## Use remote code storage

In addition to storing your  code in a Docker image, Prefect also supports deploying your code to remote storage. This approach allows you to 
store your flow code in a remote location, such as a Git repository or cloud storage service.

Using remote storage for your code has several advantages:

1. Faster iterations: You can update your flow code without rebuilding Docker images.
2. Reduced storage requirements: You don't need to store large Docker images for each code version.
3. Flexibility: You can use different storage backends based on your needs and infrastructure.

Using an existing remote Git repository like GitHub, GitLab, or Bitbucket works really well as remote code storage because:

1. Your code is already there.
2. You can deploy to multiple environments via branches and tags.
3. You can roll back to previous versions of your flows.

To deploy using remote storage, you'll need to specify where your code is stored by using `flow.from_source` to first load your flow code from a remote location.

Here's an example of loading a flow from a Git repository and deploying it:

```python git-deploy.py
from prefect import flow


if __name__ == "__main__":
    flow.from_source(
        source="https://github.com/username/repository.git",
        entrypoint="path/to/your/flow.py:your_flow_function"
    ).deploy(
        name="my-deployment",
        work_pool_name="my-work-pool",
    )
```

The `source` parameter can accept a variety of remote storage options including:

- Git repositories
- S3 buckets (using the `s3://` scheme)
- Google Cloud Storage buckets (using the `gs://` scheme)
- Azure Blob Storage (using the `az://` scheme)

The `entrypoint` parameter is the path to the flow function within your repository combined with the name of the flow function.

Learn more about remote code storage [here](https://docs.prefect.io/v3/deploy/infrastructure-concepts/store-flow-code/).

## Set default parameters

You can set default parameters for a deployment using the `parameters` keyword argument in `flow.deploy`.

```python default-parameters.py
from prefect import flow


@flow
def my_flow(name: str = "world"):
    print(f"Hello, {name}!")


if __name__ == "__main__":
    my_flow.deploy(
        name="my-deployment",
        work_pool_name="my-work-pool",
        # Will print "Hello, Marvin!" by default instead of "Hello, world!"
        parameters={"name": "Marvin"},
        image="my-docker-image:dev",
        push=False,
    )
```

Note these parameters can still be overridden on a per-deployment basis.

## Set job variables

You can set default job variables for a deployment using the `job_variables` keyword argument in `flow.deploy`. 
The job variables provided will override the values set on the work pool.

```python job-variables.py
import os
from prefect import flow


@flow
def my_flow():
    name = os.getenv("NAME", "world")
    print(f"Hello, {name}!")


if __name__ == "__main__":
    my_flow.deploy(
        name="my-deployment",
        work_pool_name="my-work-pool",
        # Will print "Hello, Marvin!" by default instead of "Hello, world!"
        job_variables={"env": {"NAME": "Marvin"}},
        image="my-docker-image:dev",
        push=False,
    )
```

Job variables can be used to customize environment variables, resources limits, and other infrastructure options, allowing fine-grained control over your infrastructure on a per-deployment or per-flow-run basis.
Any variable defined in the base job template of the associated work pool can be overridden by a job variable.

You can learn more about job variables [here](https://docs.prefect.io/v3/deploy/infrastructure-concepts/customize).

## Deploy multiple flows

To deploy multiple flows at once, use the `deploy` function.

```python multi-deploy.py
from prefect import flow, deploy


@flow
def my_flow_1():
    print("I'm number one!")


@flow
def my_flow_2():
    print("Always second...")


if __name__ == "__main__":
    deploy(
        # Use the `to_deployment` method to specify configuration 
        #specific to each deployment
        my_flow_1.to_deployment("my-deployment-1"),
        my_flow_2.to_deployment("my-deployment-2"),

        # Specify shared configuration for both deployments
        image="my-docker-image:dev",
        push=False,
        work_pool_name="my-work-pool",  
    )
```


When we run the above script, it will build a single Docker image for both deployments.
This approach offers the following benefits:

    - Saves time and resources by avoiding redundant image builds.
    - Simplifies management by maintaining a single image for multiple flows.

## Additional resources

- [Work Pools](https://docs.prefect.io/v3/deploy/infrastructure-concepts/work-pools/)
- [Store Flow Code](https://docs.prefect.io/v3/deploy/infrastructure-concepts/store-flow-code/)
- [Customize Infrastructure](https://docs.prefect.io/v3/deploy/infrastructure-concepts/customize/)
- [Schedules](https://docs.prefect.io/v3/automate/add-schedules/)
- [Write Flows](https://docs.prefect.io/v3/develop/write-flows/)