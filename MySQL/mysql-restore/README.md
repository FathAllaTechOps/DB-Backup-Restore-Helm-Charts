# MySQL Restore Job Helm Chart

This Helm chart deploys a Kubernetes Job for restoring a MySQL database from a backup stored in an S3 bucket.

## Installation and Deployment

1. Add the repository containing the Helm chart:
```
helm repo add my-repo https://path/to/repo
```

2. Install or upgrade the Helm chart:
```
helm upgrade --install mysql-restore my-repo/mysql-restore -f values.yaml
```

**Note:** Replace `values.yaml` with the appropriate values file for your environment (e.g., `values-dev.yaml` or `values-prod.yaml`).

## Prerequisites

Before deploying the MySQL Restore Helm chart, ensure the following prerequisites are met:

1. **S3 Bucket**: Create an Amazon S3 bucket to store the backups. Note down the bucket name and region for configuration.
   
2. **EKS Service Account**: Create a Kubernetes ServiceAccount in your EKS cluster with the necessary permissions to access the S3 bucket. Ensure that the ServiceAccount has permissions to read and write to the S3 bucket.
   
   Example IAM policy for the ServiceAccount:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:PutObject"
         ],
         "Resource": [
           "arn:aws:s3:::<bucket-name>/*"
         ]
       }
     ]
   }

## Configuration

The following table lists the configurable parameters and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.enabled` | Enable or disable the ServiceAccount creation | `true` |
| `serviceAccount.name` | Name of the ServiceAccount | `s3-restore-sa` |
| `bucket.name` | Name of the S3 bucket for storing backups | `<Bucket name>` |
| `bucket.prefix` | Prefix for the backup files in the S3 bucket | `"mysql-backups"` |
| `bucket.region` | AWS region of the S3 bucket | `"eu-west-1"` |
| `bucket.file` | Name of the backup file in the S3 bucket | `<File name>` |
| `database.host` | Hostname or IP address of the database server | `<Host URL>` |
| `database.port` | Port number of the database server | `3306` |
| `database.name` | Name of the database to restore | `<Database name>` |
| `database.username` | Username for connecting to the database | `<Username>` |
| `database.secretName` | Name of the Kubernetes Secret containing the database password | `<Secret name>` |
| `notifications.enabled` | Enable or disable notifications | `true` |
| `notifications.method` | Notification method (e.g., teams, email) | `"teams"` |
| `notifications.teamsWebhookUrl` | Microsoft Teams webhook URL for notifications | `<Webhook URL>` |
| `project.name` | Name of the project (optional) | `"4UM"` |
| `confluence.enabled` | Enable or disable Confluence integration | `true` |
| `confluence.pageTitle` | Title of the Confluence page for backup history | `<Page title>` |
| `confluence.pageId` | ID of the Confluence page for backup history | `<Page ID>` |
| `confluence.confluenceUrl` | URL of the Confluence API | `<Confluence URL>` |

Customize the values in the `values.yaml` file according to your environment and requirements.

## Usage

The MySQL Restore Job will start automatically once the Helm chart is deployed. To monitor the status of the job, use the following command:

```
kubectl get job mysql-restore
```

You can also view the logs of the pod to troubleshoot any issues:
```
kubectl logs <pod-name>
```

## License
This Helm chart is distributed under the Apache License 2.0.
