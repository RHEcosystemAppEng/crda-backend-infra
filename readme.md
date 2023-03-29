# CRDA backend infrastructure

## Overview

This repository contains the code for deploying the backend infrastructure, with the exception of the OpenShift cluster and ML models.

CRDA uses the following components to deploy the backend

| Component | Sub-component | Used infra | Additional info |
|---|---|---|---|
| 3Scale |  | Managed by SRE team |  |
| API Server |  | OpenShift |  |
| PgBouncer |  | OpenShift |  |
| Backbone |  | OpenShift |  |
| GraphDB | Gremlin | OpenShift |  |
|  | JanusGraph | OpenShift<br>AWS DynamoDB | AWS DynamoDB as a storage backend|
| License Analytics service |  | OpenShift |  |
| Recommendation Engine Service | Vulnerabilities | OpenShift | The data consumed from JanusGraph |
|  | Dependencies | OpenShift | The data fetched from JanusGraph, GitHub |
|  | Licenses | OpenShift | The data consumed from JanusGraph |
|  | Recommendation | OpenShift<br>AWS S3 | The data fetched from ML models stored in AWS S3 |
| Release monitors |  | OpenShift | The data fetched from package repositories (NPM/PyPI) |
| EPV Ingestion Service |  | OpenShift<br>AWS SQS | Push the data to SQS queue |
| Ingestion workers |  | OpenShift<br>AWS SQS | Listen to SQS queueThe metadata is saved in RDS table and generated files saved in S3 buckets |
| Graph Sync |  | OpenShift<br>AWS RDS<br>AWS S3 | The data fetched from PostgreSQL and AWS S3 |
| Snyk monitors |  | OpenShift<br>AWS S3 | The data fetched from Snyk API and saved to AWS S3 |
| GitHub Refresh |  | OpenShift |  |
| Unknown EPV Ingestion Service |  | OpenShift |  |
| PostgreSQL |  | AWS RDS | One database with 19 tables |
| Message queue |  | AWS SQS | 39 queues for every type of Celery task.<br> Created automatically by the service `fabric8-analytics-worker` |
| NoSQL database |  | AWS DynamoDB | One instance with 6 tables for edges, index, logs etc.<br> Created automatically by the service `fabric8-gremlin-server`|
| ML models |  | Google BigQuery<br>AWS EMR<br>AWS S3 |  |

## Prerequisites

- AWS account with permissions to manage S3 buckets and RDS databases.
- The `AWS_PROFILE` [environment variable](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html#using-profiles) is used to install components.
- `AWS_PROFILE` environment variable must be configured inside `virtualenv`.
- By default `AWS_REGION` is set up to `eu-north-1`. To change the region, configure the paramater `aws_region` in the `resources/ansible/vars/common.yml` file.

### Configuring virtualenv

```shell
python -m pip install --user virtualenv
virtualenv resources/ansible/infra-deploy
source resources/ansible/infra-deploy/bin/activate
pip install ansible boto3 botocore jmespath
```

### Configuring AWS profile

```shell
export AWS_PROFILE=<AWS_PROFILE_NAME>
```

## Infrastructure deployment

### S3 buckets

```shell
ansible-playbook resources/ansible/playbooks/s3/s3.yml
```

### RDS database

Create the master username and master password via `ansible-vault`

```shell
ansible-vault create vault resources/ansible/rds/vault
```

Add the following values to the vault file:

```shell
master_username: coreapi
master_user_password: <MASTER_PASSWORD>
```

Install RDS instance

```shell
ansible-playbook --ask-vault-pass resources/ansible/playbooks/rds/rds.yml
```