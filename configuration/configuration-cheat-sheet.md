# SonarQube Server configuration cheat sheet

This page contains the essential configuration steps needed to get started with SonarQube Server. The configuration process is divided into three main phases:

- [Before First Startup](#before-first-startup) - Essential properties before first startup
- [After First Startup](#after-first-startup) - Optional properties after first startup  
- [UI Configuration](#ui-configuration) - UI-based configuration

### Important Notes

#### Deployment Methods
This guide focuses on `sonar.properties` configuration relevant to zip file installation on a VM. However, the same configuration steps apply to other deployment methods:

- **Docker**: Configuration parameters are set using corresponding environment variables (e.g., `sonar.jdbc.url` becomes `SONAR_JDBC_URL`). See the [official documentation](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/system-properties/common-properties/) for details.
- **Kubernetes**: Parameters should be set in the `sonar.properties` section of your deployment configuration.

For more detail, see [System properties configuration methods](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/system-properties/configuration-methods/).

#### UI Configuration Automation
While this guide covers manual UI configuration, note that these settings can also be automated using the SonarQube Server API.

#### Data Center Edition
The configuration options in the `sonar.properties` file in this guide apply to the Data Center Edition of SonarQube Server, but additional configuration items are needed for such deployments beyond what's covered here.

## Before First Startup

This section covers the essential `sonar.properties` configuration that **must** be completed before starting SonarQube Server for the first time.

### Required sonar.properties Configuration

The only essential configuration before first startup is establishing the connection to an external database. This requires configuring three key parameters:

#### Database Connection Parameters

- **`sonar.jdbc.url`** - The JDBC URL that specifies the database connection string, including the database server location, port, and database name.

- **`sonar.jdbc.username`** - The database username for authentication. Only required when using SQL user authentication.

- **`sonar.jdbc.password`** - The database password for authentication. Only required when using SQL user authentication.

**Note:** The username and password parameters are only needed when using SQL user authentication. Alternative authentication methods (such as integrated authentication on Windows or certificate-based authentication) may not require these credentials.

## After First Startup

This section covers optional `sonar.properties` configuration that can be added after the initial startup to fine-tune your SonarQube Server installation. These changes require a server restart.

### Optional sonar.properties Configuration

[Content to be added]

## UI Configuration

This section covers the configuration of various settings through the SonarQube Server web interface after the server is running.

### UI Configuration Steps

[Content to be added]
