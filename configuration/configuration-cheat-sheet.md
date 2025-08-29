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

If SonarQube Server successfully connects to the database, it generally starts without issues. The UI will then be accessible on port 9000 (default) and it will be possible to use with the default administration account: username `admin`, password `admin`. The password must be changed on the first login.

## After First Startup

This section covers optional `sonar.properties` configuration that can be added after the initial startup to fine-tune your SonarQube Server installation. **These changes require a server restart.**

### Optional sonar.properties Configuration

#### Access Log Pattern for Reverse Proxy

If SonarQube runs behind a reverse proxy (strongly recommended), configure the access log pattern to display the correct remote IP address:

```properties
sonar.web.accessLogs.pattern=%i{X-Forwarded-For} %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
```

#### Performance Related Parameters

On larger, enterprise instances, the default JVM configuration is usually insufficient. The following parameters must be uncommented and updated:

- **`sonar.web.javaOpts`** - JVM options for the web server process
- **`sonar.ce.javaOpts`** - JVM options for the compute engine process  
- **`sonar.search.javaOpts`** - JVM options for the search process

**Example:** To increase the heap space allocated to the compute engine process, increase the `sonar.ce.javaOpts` from `-Xmx2g` to `-Xmx4g` to allow running with 2 compute engine workers.

#### HTTP Proxy Configuration

If a proxy is required for SonarQube Server to reach external services, configure these parameters:

- `http.proxyHost`
- `http.proxyPort`
- `https.proxyHost`
- `https.proxyPort`
- `http.auth.ntlm.domain`
- `http.proxyUser`
- `http.proxyPassword`
- `http.nonProxyHosts`

#### LDAP Authentication

If LDAP authentication is to be configured, set the security realm and configure LDAP parameters:

```properties
sonar.security.realm=LDAP
```

Then configure the relevant `ldap.*` parameters. See the [LDAP documentation](https://docs.sonarsource.com/sonarqube-server/latest/instance-administration/authentication/ldap/) for more details.

## UI Configuration

This section covers the configuration of various settings through the SonarQube Server web interface after the server is running.

### UI Configuration Steps

All configuration in this section is performed on the Administration page and requires **global administrator** privileges.

**Note:** All configuration fields in the UI are searchable, making it easy to find specific settings.

#### Configuration > General Settings > General

- **Set server base URL**: In the General tab, configure "Server base URL" (`sonar.core.serverBaseURL`)
- **Change default branch name**: If your organization doesn't default to `main`, update the "Default main branch name" (`sonar.projectCreation.mainBranchName`)
- **(Optional) Disable inherited rules deactivation**: Disable "Enable deactivation of inherited rules" option. This is the safer option early in your SonarQube journey, as it prevents users with 'Administer Quality Profiles' permission from deactivating inherited rules in quality profiles.

#### Configuration > General Settings > Security

- **(Optional) Set token lifetime**: Configure "Maximum allowed lifetime" for tokens to prevent users from having tokens that never expire
- **(Optional) Restrict project permission management**: Disable "Enable permission management for project administrators" if you plan on having tight control over project permissions. Once disabled, only global administrators can control project-level permissions
- **Never disable "Force user authentication"** - This should always remain enabled

#### Configuration > General Settings > New Code

- **Set default New Code configuration**: Set to "Number of days" and configure to 30 days. This means that by default, the New Code will be set to the last three weeks on all projects (can be changed per project as needed)

#### Configuration > General Settings > AI Code Fix

- **Enable AI Code Fix**: Review the [Terms and Conditions](https://www.sonarsource.com/legal/ai-codefix-terms/) first, then enable this feature

#### Projects > Management

- **Set default project visibility**: Configure default project visibility to "Private" (important for security and access control)

#### Projects > Background Tasks

- **Configure compute engine workers**: If you provisioned additional computing resources and configured the JVM parameters appropriately, adjust the number of compute engine workers. See the [Performance Related Parameters](#performance-related-parameters) section for JVM configuration details.

### Configuring Integrations with External Systems

#### Email Notifications Configuration

Configure email notifications on **Administration > Configuration > General Settings > Email Notification**.

#### Authentication

Configure external authentication providers on **Administration > Configuration > General Settings > Authentication**.

Available authentication options:
- **SAML** - Single Sign-On authentication
- **GitHub** - GitHub OAuth authentication
- **GitLab** - GitLab OAuth authentication  
- **Bitbucket** - Bitbucket OAuth authentication

#### DevOps Platform Integration

Configure DevOps platform integrations on **Administration > Configuration > General Settings > DevOps Platform Integrations**.

Available platform integrations:
- **GitHub** - GitHub integration for pull request decoration and repository onboarding
- **GitLab** - GitLab integration for merge request decoration and repository onboarding
- **Bitbucket** - Bitbucket Server and Cloud integration for pull request decoration and repository onboarding
- **Azure DevOps** - Azure DevOps integration for pull request decoration and repository onboarding

## Next Steps

After completing the basic configuration of your SonarQube Server, the next steps involve:

- **Permission and Access Management** - Configure user permissions and access controls
- **Project Onboarding and Analysis** - Set up projects for analysis and onboard development teams

**Note:** These topics are beyond the scope of this configuration cheat sheet.
