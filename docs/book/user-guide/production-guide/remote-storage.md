---
description: Transitioning to remote artifact storage.
---

# Chapter 3: Connecting Remote Storage

In the previous chapters, we've been working with artifacts stored locally on our machines. This setup is fine for individual experiments, but as we move towards a collaborative and production-ready environment, we need a solution that is more robust, shareable, and scalable. Enter remote storage!

Remote storage allows us to store our artifacts in the cloud, which means they're accessible from anywhere and by anyone with the right permissions. This is essential for team collaboration and for managing the larger datasets and models that come with production workloads.

When using a stack with remote storage, nothing changes except the fact that the artifacts
get materialized in a central and remote storage location. This diagram explains the flow:

<figure><img src="../../.gitbook/assets/local_run_with_remote_artifact_store.png" alt=""><figcaption><p>Sequence of events that happen when running a pipeline on a remote artifact store.</p></figcaption></figure>

## Provisioning and registering a remote artifact store

Out of the box, ZenML ships with [many different supported artifact store flavors](../../stacks-and-components/component-guide/artifact-stores/). For convenience, here are some brief instructions on how to quickly get up and running on the major cloud providers:

{% tabs %}
{% tab title="AWS" %}
You will need to install and set up the AWS CLI on your machine as a
prerequisite, as covered in [the AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html), before
you register the S3 Artifact Store.

The Amazon Web Services S3 Artifact Store flavor is provided by the [S3 ZenML integration](../../stacks-and-components/component-guide/artifact-stores/s3.md), you need to install it on your local machine to be able to register an S3 Artifact Store and add it to your stack:

```shell
zenml integration install s3 -y
```

{% hint style="info" %}
Having trouble with this command? You can use `poetry` or `pip` to install the requirements of any ZenML integration directly. In order to obtain the exact requirements of the AWS S3 integration you can use `zenml integration requirements s3`.
{% endhint %}

The only configuration parameter mandatory for registering an S3 Artifact Store is the root path URI, which needs to point to an S3 bucket and take the form `s3://bucket-name`. In order to create a S3 bucket, refer to the [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html).

With the URI to your S3 bucket known, registering an S3 Artifact Store can be done as follows:

```shell
# Register the S3 artifact-store
zenml artifact-store register cloud_artifact_store -f s3 --path=s3://bucket-name
```

For more information, read the [dedicated S3 artifact store flavor guide](../../stacks-and-components/component-guide/artifact-stores/s3.md).
{% endtab %}
{% tab title="GCP" %}
You will need to install and set up the Google Cloud CLI on your machine as a prerequisite, as covered in [the Google Cloud documentation](https://cloud.google.com/sdk/docs/install-sdk) , before you register the GCS Artifact Store.

The Google Cloud Storage Artifact Store flavor is provided by the [GCP ZenML integration](../../stacks-and-components/component-guide/artifact-stores/gcp.md), you need to install it on your local machine to be able to register a GCS Artifact Store and add it to your stack:

```shell
zenml integration install gcp -y
```

{% hint style="info" %}
Having trouble with this command? You can use `poetry` or `pip` to install the requirements of any ZenML integration directly. In order to obtain the exact requirements of the GCP integrations you can use `zenml integration requirements gcp`.
{% endhint %}

The only configuration parameter mandatory for registering a GCS Artifact Store is the root path URI, which needs to point to a GCS bucket and take the form `gs://bucket-name`. Please
read [the Google Cloud Storage documentation](https://cloud.google.com/storage/docs/creating-buckets) on how to provision a GCS bucket.

With the URI to your GCS bucket known, registering a GCS Artifact Store can be done as follows:

```shell
# Register the GCS artifact store
zenml artifact-store register cloud_artifact_store -f gcp --path=gs://bucket-name
```

For more information, read the [dedicated GCS artifact store flavor guide](../../stacks-and-components/component-guide/artifact-stores/gcp.md).
{% endtab %}
{% tab title="Azure" %}
You will need to install and set up the Azure CLI on your machine as a prerequisite, as covered in [the Azure documentation](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli), before you register the Azure Artifact Store.

The Microsoft Azure Artifact Store flavor is provided by the [Azure ZenML integration](../../stacks-and-components/component-guide/artifact-stores/azure.md), you need to install it on your local machine to be able to register an Azure Artifact Store and add it to your stack:

```shell
zenml integration install azure -y
```

{% hint style="info" %}
Having trouble with this command? You can use `poetry` or `pip` to install the requirements of any ZenML integration directly. In order to obtain the exact requirements of the Azure integration you can use `zenml integration requirements azure`.
{% endhint %}

The only configuration parameter mandatory for registering an Azure Artifact Store is the root path URI, which needs to
point to an Azure Blog Storage container and take the form `az://container-name` or `abfs://container-name`. Please
read [the Azure Blob Storage documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal)
on how to provision an Azure Blob Storage container.

With the URI to your Azure Blob Storage container known, registering an Azure Artifact Store can be done as follows:

```shell
# Register the Azure artifact store
zenml artifact-store register cloud_artifact_store -f azure --path=az://container-name
```

For more information, read the [dedicated Azure artifact store flavor guide](../../stacks-and-components/component-guide/artifact-stores/azure.md).
{% endtab %}
{% tab title="Other" %}
You can create a remote artifact store in pretty much any environment, including other cloud providers using a cloud-agnostic artifact storage such as [Minio](../../stacks-and-components/component-guide/artifact-stores/artifact-stores.md).

It is also relatively simple to create a [custom stack component flavor](../../stacks-and-components/custom-solutions/implement-a-custom-stack-component.md) for your use case.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Having trouble with setting up infrastructure? Join the [ZenML community](https://zenml.io/slack) and ask for help!
{% endhint %}

## Configuring permissions with your first service connector

While you can go ahead and [run your pipeline on your stack](#running-a-pipeline-on-a-cloud-stack) if your local client is configured to access it, it is best practice to use a [service connector](../../stacks-and-components/auth-management/auth-management.md) for this purpose. Service connectors are quite a complicated concept (We have a whole [docs section](../../stacks-and-components/auth-management/auth-management.md) on them) - but we're going to be starting with a very basic approach.

First, let's understand what a service connector does. In simple words, a service connector contains credentials that grant stack components access to cloud infrastructure. These credentials are stored in the form a [secret](../advanced-guide/secret-management/secret-management.md), and are available to the ZenML server to use. Using these credentials, the service connector brokers a short-lived token and grants temporary permissions to the stack component to access that infrastructure. This diagram represents this process:

<figure><img src="../../.gitbook/assets/ConnectorsDiagram.png" alt=""><figcaption><p>Service Connectors abstract away complexity and implement security best practices</p></figcaption></figure>

There are [many ways of creating a service connector](../../stacks-and-components/auth-management/auth-management.md), but the easiest way is to use the `auto-configure`:

{% tabs %}
{% tab title="AWS" %}
```shell
zenml service-connector register local_service_connector --type aws --auto-configure
```
{% hint style="info" %}
Having trouble with this command? You can read more about the AWS service connectors [here](../../stacks-and-components/auth-management/aws-service-connector.md).
{% endhint %}
{% endtab %}
{% tab title="GCP" %}
```shell
zenml service-connector register local_service_connector --type gcp --auto-configure
```
{% hint style="info" %}
Having trouble with this command? You can read more about the GCP service connectors [here](../../stacks-and-components/auth-management/gcp-service-connector.md).
{% endhint %}
{% endtab %}
{% tab title="Azure" %}
```shell
zenml service-connector register local_service_connector --type azure --auto-configure
```
{% hint style="info" %}
Having trouble with this command? You can read more about the Azure service connectors [here](../../stacks-and-components/auth-management/azure-service-connector.md).
{% endhint %}
{% endtab %}
{% endtabs %}

The auto-configure mechanism simply copies your local client credentials and configures a service connector. Therefore, your local credentials
should have permission to access the artifact store defined in the previous step.

Once we have our service connector, we can now attach it to stack components. In this case, we are going to connect it to our remote artifact store:

```shell
zenml artifact-store connect cloud_artifact_store --connector aws-generic
```

Now, every time you (or anyone else with access) uses the `cloud_artifact_store`, they will be granted a temporary token that will grant them access to the remote storage. Therefore, your colleagues don't need to worry about setting up credentials and installing clients locally!

## Running a pipeline on a cloud stack

Now that we have our remote artifact store registered, we can [register a new stack](understand-stacks.md#registering-a-stack) with it, just like we did in the previous chapter:

{% tabs %}
{% tab title="CLI" %}
```shell
zenml stack register local_with_remote_storage -o default -a cloud_artifact_store
```
{% endtab %}
{% tab title="Dashboard" %}
<figure><img src="../../.gitbook/assets/CreateStack.png" alt=""><figcaption><p>Register a new stack.</p></figcaption></figure>
{% endtab %}
{% endtabs %}

Now, using the [code from the previous chapter](understand-stacks.md#run-a-pipeline-on-the-new-local-stack), we run a training
pipeline:

Set our `local_with_remote_storage` stack active:

```shell
zenml stack set local_with_remote_storage
```

Let us continue with the example from the previous page and run the training pipeline:
```shell
python run.py --training-pipeline
```

When you run that pipeline, ZenML will automatically store the artifacts in the specified remote storage, ensuring that they are preserved and accessible for future runs and by your team members. You can ask your colleagues to connect to the same [ZenML server](deploying-zenml.md), and you will notice that
if they run the same pipeline, the pipeline would be partially cached, **even if they have not run the pipeline themselves before**. 

You can list your artifact versions as follows:


{% tabs %}
{% tab title="CLI" %}
```shell
# This will give you the artifacts from the last 15 minutes
zenml artifact version list --created="gte:$(date -d '15 minutes ago' '+%Y-%m-%d %H:%M:%S')"
```
{% endtab %}
{% tab title="Cloud Dashboard" %}
The [ZenML Cloud](https://zenml.io/cloud) features an [Artifact Control Plane](../starter-guide/manage-artifacts.md) to visualize artifact versions:
<figure><img src="../../.gitbook/assets/dcp_artifacts_versions_list.png" alt=""><figcaption><p>See artifact versions in the cloud.</p></figcaption></figure>
{% endtab %}
{% endtabs %}

You will notice above that some artifacts are stored locally, while others are stored in a remote storage location.

By connecting remote storage, you're taking a significant step towards building a collaborative and scalable MLOps workflow. Your artifacts are no longer tied to a single machine but are now part of a cloud-based ecosystem, ready to be shared and built upon.

<!-- For scarf -->
<figure><img alt="ZenML Scarf" referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=f0b4f458-0a54-4fcd-aa95-d5ee424815bc" /></figure>