# Connect SonarQube to an external database

One of the first configuration items in the SonarQube Helm chart is the connection to an external database that will be used by the SonarQube Server application. This page explains this configuration.

> [!IMPORTANT]  
> By default, the SonarQube Helm chart installs a PostgreSQL database from an embedded Helm chart. **Using this feature is <u>not</u> recommended!** A production installation should **always** rely on an external database.

In addition to the above note, the embedded PostgreSQL Helm chart is now deprecated. It has historically been included for convenience and to allow quick tests. However, any meaningful tests must be done with an external DB.

## Steps to enable external DB connection:
- Set `postgresql.enabled` to `false`
- Set `jdbcOverwrite.enabled` to `true`
- Configure the `jdbcOverwrite.jdbcUrl`
  - For example: *jdbc:postgresql://myPostgres/myDatabase*
- Set the `jdbcOverwrite.jdbcUsername`
  - This is the username of the database user that SonarQube will use to connect to the database.
  - For example: *sonar*
- Set the `jdbcOverwrite.jdbcPassword`
  - The password of the DB user.
  - For example: *sonarPass*

The `jdbcOverwrite.jdbcPassword` is currently deprecated and should only be used when getting started with the Helm chart. See the next section.

## DB connection parameters set in secrets

The above section can be used as the first step when getting started with the Helm chart. However, the database connection parameters (at least the password) should be set in secrets so that they are not stored in the Helm chart in plain text.

Set the database password from a secret:
- Create the secret. This can be done in different ways. Here is an example `kubectl` command:
  ```
  kubectl create secret generic sonar-database --from-literal=sonar-database-password=sonarPass -n sonarqube
  ```
  - Parameters in the above command:
    - `sonar-database` is the name of the secret
    - `sonar-database-password` is the key of the database password stored in the secret
    - `sonarPass` is the password as it was used in the example above
    - `sonarqube` is the namespace where the SonarQube Helm chart is installed.
- Comment out the `jdbcOverwrite.jdbcPassword` parameter. Do not set it anymore.
- Set the `jdbcSecretName`
  - For example: *sonar-database* from the example command above.
- Set the `jdbcSecretPasswordKey`
  - For example: *sonar-database-password* from the example command above.

## Set the user name and URL from a ConfigMap

The database username and the JDBC URL values can be set in a ConfigMap (or a secret). In some cases, it may be cumbersome to store these values in the values file as they are only known at deployment time. In such cases, the values can be read from ConfigMaps or secrets. The parameters need to be stored in the environment variable format as documented [here](https://docs.sonarsource.com/sonarqube-server/latest/setup-and-upgrade/environment-variables/).

For the database connection specifically, the relevant environment variables are `SONAR_JDBC_URL` and `SONAR_JDBC_USERNAME`. The corresponding ConfigMap definition may then look like this:
```
kubectl create configmap sonar-configmap -n sonarqube \
   --from-literal=SONAR_JDBC_URL=jdbc:postgresql://myPostgres/myDatabase \
   --from-literal=SONAR_JDBC_USERNAME=sonarqube
```

Then, the ConfigMap must be referenced in the Helm chart's `extraConfig.configmaps` parameter:
```
extraConfig:
  secrets: []
  configmaps:
    - sonar-configmap
```

> [!TIP]
> This approach can be used to set any of the documented environment variables mentioned on [this docs page](https://docs.sonarsource.com/sonarqube-server/latest/setup-and-upgrade/environment-variables/).