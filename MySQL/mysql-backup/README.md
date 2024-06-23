# MySQL Backup Helm Chart

This Helm chart deploys a CronJob that performs automated backups of a MySQL database to an Amazon S3 bucket. The backups are scheduled to run at a specified time (e.g., daily at 7 AM UTC), and notifications are sent to a Microsoft Teams webhook with the backup status and details.

## Prerequisites

Before deploying the MySQL Backup Helm chart, ensure the following prerequisites are met:

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


## Installation and Deployment

1. Add the repository containing the Helm chart:
```
helm repo add my-repo https://path/to/repo
```

2. Install or upgrade the Helm chart:
```
helm upgrade --install mysql-backup my-repo/mysql-backup -f values.yaml
```


**Note:** Replace `values.yaml` with the appropriate values file for your environment (e.g., `values-dev.yaml` or `values-prod.yaml`).

## Configuration

The following table lists the configurable parameters and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.enabled` | Enable or disable the ServiceAccount creation | `true` |
| `serviceAccount.name` | Name of the ServiceAccount | `s3-backup-sa` |
| `serviceAccount.annotations` | Annotations for the ServiceAccount (e.g., IAM role ARN) | `{}` |
| `bucket.name` | Name of the S3 bucket for storing backups | `""` |
| `bucket.prefix` | Prefix for the backup files in the S3 bucket | `"mysql-backups"` |
| `bucket.region` | AWS region of the S3 bucket | `""` |
| `database.type` | Type of the database (e.g., MySQL, PostgreSQL) | `"MySQL"` |
| `database.host` | Hostname or IP address of the database server | `""` |
| `database.port` | Port number of the database server | `3306` |
| `database.name` | Name of the database to backup | `""` |
| `database.username` | Username for connecting to the database | `""` |
| `database.secretName` | Name of the Kubernetes Secret containing the database password | `""` |
| `backup.schedule` | Cron schedule for the backup job | `"0 7 * * *"` |
| `notifications.enabled` | Enable or disable notifications | `true` |
| `notifications.method` | Notification method (e.g., teams, email) | `"teams"` |
| `notifications.teamsWebhookUrl` | Microsoft Teams webhook URL for notifications | `""` |
| `project.name` | Name of the project (optional) | `""` |
| `confluence.enabled` | Enable or disable Confluence integration | `false` |
| `confluence.pageTitle` | Title of the Confluence page for backup history | `""` |
| `confluence.pageId` | ID of the Confluence page for backup history | `""` |
| `confluence.confluenceUrl` | URL of the Confluence API | `""` |

Customize the values according to your environment and requirements.

## Monitoring and Logs

To check the status of the deployed CronJob:
```
kubectl get cronjob mysql-backup -n <namespace>
```


To view the logs of the most recent backup job:
```
kubectl logs -l job-name=$(kubectl get jobs --sort-by=.metadata.creationTimestamp -o jsonpath='{.items[-1].metadata.name}') -c backup -n <namespace>
```

To manually trigger a backup job for testing:
```
kubectl create job --from=cronjob/mysql-backup manual-backup-$(date +%s) -n <namespace>
```