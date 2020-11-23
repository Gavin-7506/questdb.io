---
title: Deployment
description: Instructions and guides showing how to deploy QuestDB.
---

## AWS AMI

You can deploy QuestDB using an Amazon Machine Image (AMI) on your EC2
instances. To facilitate the process, we maintain a
[repository]({@githubOrgUrl@}/questdb-packer-ami) with a
[Packer](https://www.packer.io) template that you can follow to create your own
AMIs.

### Prerequisites

In order to use this template, you will need:

- [Packer](https://www.packer.io/docs/install/index.html)
- [Template]({@githubOrgUrl@}/questdb-packer-ami) from our GitHub repository
- AWS credentials

### Build the AMI

1. Clone the repository:

```bash
git clone https://github.com/questdb/questdb-packer-ami.git
```

2. Navigate to `src` and run `packer`:

```bash
cd questdb-packer-ami/src
packer build template.json
```

### Usage

#### Configuration

The AMI built using the template is generic. You might want to change the
database configuration file:

```
src/config/server.conf
```

For all the properties and values that you can set, please check the
[configuration page](https://questdb.io/docs/reference/configuration).

#### Logs and AWS CloudWatch

The AMI uses [logrotate](https://linux.die.net/man/8/logrotate) to automatically
trim and archive logs generated by QuestDB. The AWS CloudWatch
[agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
is also preinstalled and already configured. If you want the logs to be
available on your CloudWatch dashboard, you need to:

1. Make sure you run your EC2 with an instance profile that has the
   `CloudWatchAgentServerPolicy`
   [IAM policy](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent.html)
2. Create the necessary CloudWatch resources: `/questdb-<instance-id>` (with
   leading slash) log group and `questdb-<instance-id>` log stream (example of
   instance id: `i-0c1386329d00506a2`)

## Kubernetes

You can deploy QuestDB in a [Kubernetes](https://kubernetes.io) cluster using a
[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
and a
[persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).
We distribute QuestDB via [Helm](https://helm.sh) on
[ArtifactHub](https://artifacthub.io/packages/helm/questdb/questdb).

### Prerequisites

- [Helm](https://helm.sh/docs/intro/install/)
- [Kubernetes CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/)

### Get the chart

Using the Helm client, add our chart repository:

```shell
helm repo add questdb https://helm.questdb.io/
```

Then, update the index:

```shell
helm repo update
```

### Run QuestDB

Make sure you have a local cluster running:

```shell
minikube start
```

Then install the chart:

```shell
helm install my-questdb questdb/questdb
```

Finally, use the Kubernetes CLI to get the pod name:

```shell
kubectl get pods
```

Result:

| NAME         | READY | STATUS  | RESTARTS | AGE   |
| ------------ | ----- | ------- | -------- | ----- |
| my-questdb-0 | 1/1   | Running | 1        | 9m59s |

### Querying QuestDB locally

In order to run queries against your local instance of QuestDB, you can use port
forwarding:

```shell
kubectl port-forward my-questdb-0 9000
```

You can use the following ports:

- 9000: [REST API](/docs/reference/api/rest/) and
  [Web Console](/docs/reference/client/web-console/)
- 8812: [Postgres](/docs/reference/api/postgres/)
- 9009: [InfluxDB line protocol](/docs/reference/api/influxdb/)