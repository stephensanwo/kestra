# Getting Started with Kestra

## What is Kestra?

Kestra is an open-source orchestration & scheduler platform that makes it easy to build, schedule, run, and monitor complex pipelines. Kestra focuses on simplicity and scalability, so that anyone i.e; Data Engineers, Software Developers, Business Analysts, etc, can quickly and easily automate their workflows, create a schedule to run the workflows, and extend the workflows to cover more complex use cases as the business requirement changes. [This article](https://kestra.io/blogs/2022-02-01-kestra-opensource.html) discusses in detail, the motivation behind Kestra.

## Pre Requisites

Kestra is shipped with Docker, to start using Kestra, you need to have docker installed and running on your system. If you don't have Docker installed already, follow the instructions [here](https://docs.docker.com/desktop/install/mac-install/).

## Installing Kestra

On your local machine, create a directory for kestra and download the docker-compose.yml file from [docker-compose.yml](https://github.com/kestra-io/kestra/blob/develop/docker-compose.yml) by running `wget https://raw.githubusercontent.com/kestra-io/kestra/develop/docker-compose.yml` on mac and linux or create a `docker-compose.yml` file, then copy the contents into the file.

Next, in the terminal, run `docker-compose pull` in the root of your kestra project; where you downloaded the `docker-compose.yml` file. This will pull all the remote images, then run `docker-compose up -d` to build the images.

In docker desktop, you should see both `kestra` and `postgres` images running within the kestra container like below:

![Image](../assets/images/docker-start.png)

Next, in your browser, access the kestra application by navigating to `http://localhost:8080`, you should see the kestra homepage like below:

![Kestra Home](../assets/images/kestra-home.png)

## Architecture of a Kestra Flow

A flow is a [yaml](https://www.redhat.com/en/topics/automation/what-is-yaml) file where you define all your tasks in the order you want to run them. To create a basic flow three important high-level parameters/fields must be defined;

- The flow `id` - a unique ID for your flow, across your namespaces
- `namespace` - a workspace to organize similar tasks, i.e. testing flows can be organized in a test namespace, while production flows can be organized in a production namespace
- `task` - a task block that defines the task or series of tasks. The `task` block will have a set of required parameters.
  - `id` the task id
  - `type` the task type references the java class that handles the task, for example, a MongoDB task type to query a collection is `io.kestra.plugin.mongodb.Find`
  - `xxx` additional fields that depend on the task type, and are usually defined in the plugin. For example, the MongoDB task `io.kestra.plugin.mongodb.Find` requires the following extra fields to connect and perform database operations:

```yml
connection:
uri: "database_uri"
database: "my_database"
collection: "my_collection"
filter:
  _id:
  $oid: "object_id"
```

Next, we will create a simple flow consisting of tasks that print out some data to the console. Flows are created in the flow tab; click on **create** in the bottom right corner, then paste the code below in the editor:

```yml
id: sample-task
namespace: io.kestra.tests
tasks:
  - id: task1
    type: io.kestra.core.tasks.debugs.Return
    description: "Test task to print task information to the console"
    format: "This is task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}"
    level: TRACE
```

The flow `sample-task` has a single task that prints some information about the flow (flow id) and task (task id, task start timestamp). Note that we use the `io.kestra.core.tasks.debugs.Return` core plugin to enable this task. You can find more information about this plugin [here](https://kestra.io/plugins/core/tasks/debugs/io.kestra.core.tasks.debugs.Return.html)

Next, click **Save** at the bottom right corner to save the flow, then click **New Execution** to execute the flow. This will open the executions tab, and you should see a green bar on task1 within the gantt chart, you can click it to show the output. You should see the output printed based on the defined format within the task as shown below:

<video width="100%" height="100%" controls>
  <source src="../assets/videos/create-basic-flow.mov" type="video/mov">
</video>

## Orchestrating Workflows

Next, we take a look at orchestrating multiple tasks within a flow using `flowable tasks`. These are special tasks in the core plugin `io.kestra.core.tasks.flows` that can be used to orchestrate multiple tasks together in building out more complex flows. There are several flowable tasks in kestra, but for now, we will focus on two (2) very important ones; **sequential** and **parallel** tasks. There are several other flowable tasks useful for different purposes, you can read the [documentation](https://kestra.io/docs/developer-guide/flowable/) to learn more about them.

### Sequential Tasks:

This is used to run multiple tasks in sequential order, this is useful when you need to run tasks that have to wait for each other, for example, if you need to run a flow that extracts data from an object storage, processes the data, then store the structured data in a database, each of these tasks will have to wait for the other to complete.

To run a sequential task, create a new flow called `sample-sequential-task` then add the code below.

```yml
id: sample-sequential-task
namespace: io.kestra.tests
revision: 3
tasks:
  - id: sequential
    type: io.kestra.core.tasks.flows.Sequential
    tasks:
      - id: task1
        type: io.kestra.core.tasks.debugs.Return
        description: First sequential task
        format: First sequential task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
      - id: task2
        type: io.kestra.core.tasks.debugs.Return
        description: Second sequential task
        format: Second sequential task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
      - id: task3
        type: io.kestra.core.tasks.debugs.Return
        description: Third sequential task
        format: Third sequentia task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
```

In the code above, we created a sequential flowable task with the core plugin `io.kestra.core.tasks.flows.Sequential` then defined three tasks to print their details to the console in sequential order, when you run this flow, you will notice in the gantt chart, that the tasks are run one after the other as shown in the image below.

![Sequential Task](../assets/images/sequential-task.png)

### Parallel Tasks:

This task is used to run multiple tasks at once without having to wait for other tasks. This is useful when tasks can run independently of each other, for example, where you need to import data from multiple object storage, and process them, you don't need to wait for the import and data processing of data from object 1 to be completed before you start importing and processing data from object 2, you can run both tasks in parallel.

Create a new flow called `sample-parallel-task` then add the code below.

```yml
id: sample-parallel-task
namespace: io.kestra.tests
revision: 1
tasks:
  - id: parallel
    type: io.kestra.core.tasks.flows.Parallel
    tasks:
      - id: task1
        type: io.kestra.core.tasks.debugs.Return
        description: First parallel task
        format: First parallel task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
      - id: task2
        type: io.kestra.core.tasks.debugs.Return
        description: Second parallel task
        format: Second parallel task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
      - id: task3
        type: io.kestra.core.tasks.debugs.Return
        description: Third parallel task
        format: Third parallel task, task id > {{task.id}} running within flow > {{flow.id}} started on {{taskrun.startDate}}
```

In the code above, we created a parallel flowable task with the core plugin `io.kestra.core.tasks.flows.Parallel` then defined three tasks to print their details to the console in sequential order, when you run this flow, you will notice in the gantt chart, that the tasks are run at the same time as shown below:

![Parallel Task](../assets/images/parallel-task.png)

## Kestra in Action - Real-world example

Next, we will look at a real-world example of how kestra can simplify your workflows. In this example, we will create a flow that extracts data from any subreddit in `.json` format and upload it to an AWS S3 bucket.

Create a new flow named `kestra-reddit-aws-example` and paste the code below.

```yml
id: kestra-reddit-aws-example
namespace: io.kestra.tests
revision: 1
inputs:
  - type: STRING
    description: Provide the subreddit url to download current data from
    name: subRedditUrl
  - type: STRING
    description: This will be the name of the file uploaded to AWS
    name: subredditName
tasks:
  - id: downloadRedditData
    type: io.kestra.plugin.fs.http.Download
    description: Download data from reddit channel
    headers:
      user-agent: kestra-io
    uri: "{{inputs.subRedditUrl}}"
  - id: uploadToAwsS3
    type: io.kestra.plugin.aws.s3.Upload
    accessKeyId: <your aws access key id>
    bucket: <your aws bucket name>
    from: "{{outputs.downloadRedditData.uri}}"
    key: "{{inputs.subredditName}}"
    region: <your aws bucket region>
    secretKeyId: <your aws secret key id>
```

First, we added two input fields to the flow, to dynamically pass variables when running the flow, this ensures that we don't have to hardcode variables within the flow.

The first variable is the url of the subreddit `subRedditUrl`, this can be changed at run time to dynamically accept any subreddit url. The second variable is the subreddit name `subredditName` which will be passed to the AWS task as the name of the file output stored in the s3 bucket.

Next, we defined two tasks, the first is an HTTP task in the core plugin `io.kestra.plugin.fs.http.Download` that enables us to download files from any url into a kestra temporary storage directory, which can then be subsequently used within other tasks. The url of the output file can be accessed through `outputs.taskName.uri`. You can learn more about this core plugin [here](https://kestra.io/plugins/plugin-fs/tasks/http/io.kestra.plugin.fs.http.Download.html)

In the second task, we upload the downloaded file to an AWS s3 bucket, using the aws plugin `io.kestra.plugin.aws.s3.Upload`. To use this plugin, you would have to create an AWS IAM user, provide programmatic access to that user, and set permissions for AWS s3 Full Access. You can follow this [guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) to create an AWS IAM user and assign permissions. _I have created temporary credentials for this tutorial, which have been subsequently deactivated_

- Create a new IAM User
  ![Create AWS User](../assets/images/create-aws-usr.png)

- Assign AWS S3 Permissions
  ![Assign S3 Access](../assets/images/s3-access.png)

- Obtain Access Key ID and Secret
  ![AWS Access](../assets/images/aws-access.png)

Once these steps have been completed, ensure that you create a bucket in S3 to store your `.json` files, also note the AWS region as shown below:

![s3 Bucket Create](../assets/images/s3-bucket-create.png)

Next update the flow data with your AWS credentials **I am using my temporary credentials below, ensure you updated with your credentials**

```yml
id: kestra-reddit-aws-example
namespace: io.kestra.tests
revision: 1
inputs:
  - type: STRING
    description: Provide the subreddit url to download current data from
    name: subRedditUrl
  - type: STRING
    description: This will be the name of the file uploaded to AWS
    name: subredditName
tasks:
  - id: downloadRedditData
    type: io.kestra.plugin.fs.http.Download
    description: Download data from reddit channel
    headers:
      user-agent: kestra-io
    uri: "{{inputs.subRedditUrl}}"
  - id: uploadToAwsS3
    type: io.kestra.plugin.aws.s3.Upload
    accessKeyId: AKIA2RSBVZVMJCYWVSWY
    bucket: kestra-io-bucket
    from: "{{outputs.downloadRedditData.uri}}"
    key: "{{inputs.subredditName}}"
    region: eu-west-1
    secretKeyId: kb0XxkfgnuzmjEGfvQ7RiglXQd8PFRQPfvRFxaAV
```

Next, click **Execution**, this will open up a dialog box requesting to provide the parameters for our inputs, as a sample, you can paste this subreddit: `https://www.reddit.com/r/news.json` as the url and `r-news-today` as the subreddit name, then click execute. If done correctly, the tasks should run as shown below. You can then check the S3 bucket to see the data loaded.

<video width="100%" height="100%" controls>
  <source src="../assets/videos/sequential-reddit-aws-download.mov" type="video/mov">
</video>

Finally, we can explore extracting data from multiple subreddits in parallel. Create a new flow named `kestra-reddit-aws-parallel-example` and add the code below

```yml
id: kestra-reddit-aws-parallel-example
namespace: io.kestra.tests
revision: 1
inputs:
  - type: STRING
    description: Provide the first subreddit url to download current data from
    name: subRedditUrl1
  - type: STRING
    description: This will be the name of the file uploaded to AWS
    name: subredditName1
  - type: STRING
    description: Provide the second subreddit url to download current data from
    name: subRedditUrl2
  - type: STRING
    description: This will be the name of the file uploaded to AWS
    name: subredditName2
  - type: STRING
    defaults: AKIA2RSBVZVMJCYWVSWY
    name: accessKeyId
    required: false
  - type: STRING
    defaults: kb0XxkfgnuzmjEGfvQ7RiglXQd8PFRQPfvRFxaAV
    name: secretKeyId
    required: false
  - type: STRING
    defaults: eu-west-1
    name: awsRegion
    required: false
tasks:
  - id: parallel
    type: io.kestra.core.tasks.flows.Parallel
    tasks:
      - id: sequential-1
        type: io.kestra.core.tasks.flows.Sequential
        tasks:
          - id: downloadRedditData1
            type: io.kestra.plugin.fs.http.Download
            description: Download data from reddit channel
            headers:
              user-agent: kestra-io
            uri: "{{inputs.subRedditUrl1}}"
          - id: uploadToAwsS31
            type: io.kestra.plugin.aws.s3.Upload
            accessKeyId: "{{inputs.accessKeyId}}"
            bucket: kestra-io-bucket
            from: "{{outputs.downloadRedditData1.uri}}"
            key: "{{inputs.subredditName1}}"
            region: "{{inputs.awsRegion}}"
            secretKeyId: "{{inputs.secretKeyId}}"
      - id: sequential-2
        type: io.kestra.core.tasks.flows.Sequential
        tasks:
          - id: downloadRedditData2
            type: io.kestra.plugin.fs.http.Download
            description: Download data from reddit channel
            headers:
              user-agent: kestra-io
            uri: "{{inputs.subRedditUrl2}}"
          - id: uploadToAwsS3
            type: io.kestra.plugin.aws.s3.Upload
            accessKeyId: "{{inputs.accessKeyId}}"
            bucket: kestra-io-bucket
            from: "{{outputs.downloadRedditData2.uri}}"
            key: "{{inputs.subredditName2}}"
            region: "{{inputs.awsRegion}}"
            secretKeyId: "{{inputs.secretKeyId}}"
```

Note that we have abstracted the AWS credentials into global variables so that we don't have to specify them for each task, we have also created a parallel task that runs two sequential tasks concurrently and provided input parameters for both task 1 and task 2. The file download and upload for each task will remain sequential, while the process is run concurrently for both subreddits provided

When you click execute, you can provide two subreddits; `https://www.reddit.com/r/news.json` and `https://www.reddit.com/r/sports.json` in the dialog box, then execute. Take note of the gantt chart as shown below, which shows both processes running in parallel.

<video width="100%" height="100%" controls>
  <source src="../assets/videos/parallel-reddit-download.mov" type="video/mov">
</video>

## Next Steps:

In this article, you have learned how to run basic workflows, and sequential and parallel tasks, and have seen a practical example of how kestra can help automate your workflows. Kestra has more advanced capabilities, including automatic scheduling, templating, switch tasks, running python or javascript files, etc. which will be covered in subsequent tutorials. Follow this [link](https://kestra.io/docs/developer-guide/flowable/) to read the developer guide and this [link](https://kestra.io/plugins/) to view the list of all plugins available on kestra.
