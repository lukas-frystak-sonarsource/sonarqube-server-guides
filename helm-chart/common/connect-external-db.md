# Connect SonarQube to an external database

One of the first configuration items in the Sonar helm chart is the connection to an external database that will be used by the SonarQube Server application. This page explains this configuration.

> [!IMPORTANT]  
> By default, the SonarQube helm chart installs PostgreSQL database from an embedded helm chart. **Using this feature is <u>not</u> recommended!** A production installation should **always** rely on an external database.

On top of the above note, the embedded PostgreSQL helm chart is now deprecated. It has been historicaly included for convenience and to allow quick tests. However, any meaningful tests must be done with an external DB.

## Steps to enable external DB connection:
- Set `postgresql.enabled` to `false`
- Set `jdbcOverwrite.enabled` to `true`
- Configure the `jdbcOverwrite.jdbcUrl`
  - For example: *jdbc:postgresql://myPostgress/myDatabase*
- Set the `jdbcOverwrite.jdbcUsername`
  - This is the username of the database user that SonarQube will use to connect to the database.
  - For example: *sonar*
- Set the `jdbcOverwrite.jdbcPassword`
  - The password of the DB user.
  - For example: *sonarPass*
