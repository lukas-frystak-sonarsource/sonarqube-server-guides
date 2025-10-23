# Running the SonarQube Server DB Copy Tool on Kubernetes

When SonarQube Server is already running on a Kubernetes cluster and you need to migrate between different database vendors, it can be convenient to run the [SonarQube DB Copy Tool](https://docs.sonarsource.com/sonarqube-server/server-update-and-maintenance/maintenance/sonarqube-db-copy-tool) directly on the Kubernetes cluster. This environment typically already has access to the existing (*"source"*) database and can be configured to access the new (*"target"*) database. By running the DB Copy Tool directly on the cluster, you can avoid setting up a separate VM for the migration.

This guide explains how to run the SonarQube Server Database Copy Tool on Kubernetes using a Job manifest.

This page focuses only on the execution of the DB Copy Tool. For more information on the complete migration process, refer to the [Databse vendor migration plan](db-copy-plan.md).

## Prerequisites

- Source and target database configurations
- No SonarQube Server instances are connected to either database
- Access to your Kubernetes cluster
- Appropriate permissions to create secrets and jobs
- `kubectl` CLI tool installed and configured (or another tool to create Kubernetes objects)

## Kubernetes Manifest

The complete Kubernetes Job manifest for the SonarQube Server DB Copy Tool can be found in this file: [`sqs-db-copy-tool-job.yaml`](../examples/kubernetes/manifests/sqs-db-copy-tool-job.yaml). Always review the manifest before running it.

The manifest can be applied directly from its GitHub URL as shown in the example below, but it can also be downloaded locally (and customized) if needed.

## Usage

1. If you want to run the Job in a dedicated namespace, create the namespace first.

2. Create a secret with the connection parameters used by the DB Copy Tool to connect to both databases:

    ```bash
    kubectl create secret generic sqs-db-copy-params \
    --from-literal=SOURCE_DB_URL=<JDBC_URL_SOURCE_DB> \
    --from-literal=SOURCE_DB_USERNAME=<DB_USER_SOURCE_DB> \
    --from-literal=SOURCE_DB_PASSWORD=<DB_PASSWORD_SOURCE_DB> \
    --from-literal=TARGET_DB_URL=<JDBC_URL_TARGET_DB> \
    --from-literal=TARGET_DB_USERNAME=<DB_USER_TARGET_DB> \
    --from-literal=TARGET_DB_PASSWORD=<DB_PASSWORD_TARGET_DB> \
    -n <namespace>
    ```

3. Then, apply the manifest to your Kubernetes cluster:

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/lukas-frystak-sonarsource/sonarqube-server-guides/refs/heads/main/examples/kubernetes/manifests/sqs-db-copy-tool-job.yaml -n <namespace>
    ```

4. While waiting for the migration to finish, you can monitor the job progress:

    ```bash
    kubectl get jobs -n <namespace>
    kubectl logs job/sqs-db-copy-job -n <namespace>
    ```
5. Look for the message indicating that the copy has finished succesfully

    ```
    kubectl logs job/sqs-db-copy-job -n sqcopytool | grep "THE COPY HAS FINISHED SUCCESSFULLY"
    ```

## Configuration

As the database connection parameters come from a secret, there isn't anything that should need configuring in the job. The only parameters that might need adjusting are the **Resource limits**. Increase the CPU and memory limits for the container if needed.

## Troubleshooting

### Logs and Debugging

Check the job logs for detailed error messages:

```bash
kubectl logs job/sqs-db-copy-job -f -n <namespace>
```

For additional troubleshooting information, you can also check the job events:

```bash
kubectl describe job sqs-db-copy-job -n <namespace>
```

## Cleanup

The Job will be automatically removed 24 hours after finishing. However, It can also be cleared up manually:

```bash
kubectl delete job sqs-db-copy-job  -n <namespace>
kubectl delete secret sqs-db-copy-params  -n <namespace>
```

Or just delete the dedicated namespace:
```bash
kubectl delete namespace <namespace>
```
