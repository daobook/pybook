# Run flows on Prefect Managed infrastructure

<span class="badge cloud"></span> Prefect Cloud can run your flows on your behalf with Prefect Managed work pools.

Flows that run with this work pool do not require a worker or cloud provider 
account—Prefect handles the infrastructure and code execution for you.

Managed execution is a great option for users who want to get started quickly, with no infrastructure setup.

## Create a managed deployment

1. Create a new work pool of type Prefect Managed in the UI or the CLI.
   Use this command to create a new work pool using the CLI:

   ```bash
   prefect work-pool create my-managed-pool --type prefect:managed
   ```

1. Create a deployment using the flow `deploy` method or `prefect.yaml`.
   Specify the name of your managed work pool, as shown in this example that uses the `deploy` method:

   ```python managed-execution.py
   from prefect import flow

   if __name__ == "__main__":
       flow.from_source(
           source="https://github.com/prefecthq/demo.git",
           entrypoint="flow.py:my_flow",
       ).deploy(
           name="test-managed-flow",
           work_pool_name="my-managed-pool",
       )
   ```

1. With your [CLI authenticated to your Prefect Cloud workspace](https://docs.prefect.io/v3/manage/cloud/manage-users/api-keys/), run the script to create your deployment:

   ```bash
   python managed-execution.py
   ```

1. Run the deployment from the UI or from the CLI.

   This process runs a flow on remote infrastructure without any infrastructure setup, starting a worker, or requiring a cloud provider account.

## Add dependencies

Prefect can install Python packages in the container that runs your flow at runtime.
Specify these dependencies in the **Pip Packages** field in the UI, or by configuring 
`job_variables={"pip_packages": ["pandas", "prefect-aws"]}` in your deployment creation like this:

```python
from prefect import flow

if __name__ == "__main__":
    flow.from_source(
        source="https://github.com/prefecthq/demo.git",
        entrypoint="flow.py:my_flow",
    ).deploy(
        name="test-managed-flow",
        work_pool_name="my-managed-pool",
        job_variables={"pip_packages": ["pandas", "prefect-aws"]}
    )
```

Alternatively, you can create a `requirements.txt` file and reference it in your [prefect.yaml pull step](https://docs.prefect.io/v3/deploy/infrastructure-concepts/prefect-yaml#utility-steps).

## Limitations

### Concurrency and work pools

Free tier accounts are limited to:

- Maximum of 1 concurrent flow run per workspace across all `prefect:managed` pools
- Maximum of 1 managed execution work pool per workspace

Pro tier and above accounts are limited to:

- Maximum of 10 concurrent flow runs per workspace across all `prefect:managed` pools.
- Maximum of 5 managed execution work pools per workspace.

### Images

Managed execution requires that you run the official Prefect Docker image: `prefecthq/prefect:3-latest`. 
However, as noted above, you can install Python package dependencies at runtime. 
If you need to use your own image, we recommend using another type of work pool.

### Code storage

You must store flow code in an accessible remote location.
Prefect supports git-based cloud providers such as GitHub, Bitbucket, or GitLab.
Remote block-based storage is also supported, so S3, GCS, and Azure Blob are additional code storage options.

### Resources

Memory is limited to 2 GB of RAM, which includes all operations such as dependency installation. 
Maximum job run time is 24 hours.

## Usage limits

Free tier accounts are limited to 10 compute hours per workspace per month. 
Pro tier and above accounts are limited to 250 hours per workspace per month. 
You can view your compute hours quota usage on the **Work Pools** page in the UI.

## Next steps

Read more about creating deployments in [Run flows in Docker containers](https://docs.prefect.io/v3/deploy/infrastructure-examples/docker/).

For more control over your infrastructure, such as the ability to run 
custom Docker images, [serverless push work pools](https://docs.prefect.io/v3/deploy/infrastructure-examples/serverless/) 
are also a good option.