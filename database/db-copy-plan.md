# Database Vendor Migration Plan

The official [SonarQube Server DB Copy Tool documentation](https://docs.sonarsource.com/sonarqube-server/server-update-and-maintenance/maintenance/sonarqube-db-copy-tool) focuses on how to use the DB Copy Tool. This page aims to expand on the official documentation and provide an example plan for the entire migration process. The migration of a database engine/vendor is a significant task that typically requires downtime of a production SonarQube Server instance.

## DB Copy Tool Overview

The DB Copy Tool enables SonarQube Server users to move the data in their SonarQube Server database from one database engine to another. For example, it allows moving from Microsoft SQL Server to PostgreSQL.

The DB Copy Tool is a Java command line application that connects to both databases: the original/existing one (the source database) and the newly set up database relying on a different database vendor (the target database). When the tool is executed, it gets migrates table data from the source to the target database.

> [!IMPORTANT]
> When the DB Copy Tool runs, no SonarQube Server instance can be running while connected to either of the databases (source or target). This implies SonarQube Server downtime. The downtime is always minimized by thorough planning and testing and this page aims to address that.

## Migration Overview

**The database migration must be tested** in a test/lower environment. This process should be done at least once to review, understand, and fine-tune it. Then, it should be tested again to validate the process and verify its production readiness. Once fully validated, the migration can be done in production.

Below are the recommended steps to proceed with the database vendor change.

1. **Prepare test environment**: Create a representative test environment where the migration can be tested and validated.
    - SonarQube Server is installed/deployed in the same way and on the same hardware as in the production environment.
    - The database SonarQube Server uses is the same database service or a database server installed in the same way as in production. The allocated resources should also be the same.

2. **Backup production data**: Dump/backup your production database and restore it in the test environment.

3. **Deploy test instance**: Deploy SonarQube Server in the test environment, connecting to the test database set up in step 2. The deployment should be as similar as possible to the production environment.

4. **Verify the test environment installation**: Verify that the SonarQube Server instance runs without issues in the test environment. Optionally, test performance to be able to compare before/after the database change.

5. **Execute database migration**: Follow the steps for the database change in the following section.

6. **Validate migration**: Once the DB copy is complete, connect your SonarQube Server instance to the copied database and verify that is starts and runs without issues. Optionally, test the instance for performance to compare before/after the database change.


## DB Copy Tool Process

Using the Sonar DB Copy Tool involves two steps:

1. Start SonarQube Server to populate the schema on an empty database
2. Run the DB Copy Tool

These steps need to be executed from somewhere where you can install SonarQube Server and have network access to both databases. This can be done on a virtual machine, in a container, or [on a Kubernetes cluster](db-copy-tool-on-kubernetes.md). The most convenient approach will likely be 

