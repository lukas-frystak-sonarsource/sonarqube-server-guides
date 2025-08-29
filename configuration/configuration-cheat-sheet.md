# SonarQube Server Configuration Cheat Sheet

## 📋 Configuration Checklist Overview

| Phase | Items | Importance |
|-------|-------|------------|
| [Before First Startup](#-before-first-startup) | 1-3 items | ✅ Essential for startup |
| [After First Startup](#%EF%B8%8F-after-first-startup) | 6 categories | ⚙️ Production readiness |
| [UI Configuration](#%EF%B8%8F-ui-configuration) | 8 sections | 🔧 Fine-tuning & integrations |

---

## 📖 Deployment Methods Reference

This guide focuses on `sonar.properties` configuration relevant to zip file installation on a VM. However, the same configuration steps apply to other deployment methods:

- **Docker**: Configuration parameters are set using corresponding environment variables (e.g., `sonar.jdbc.url` becomes `SONAR_JDBC_URL`). See the [official documentation](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/system-properties/common-properties/) for details.
- **Kubernetes**: Parameters should be set in the `sonar.properties` section of your deployment configuration.

| Method | Configuration | Notes |
|--------|---------------|-------|
| **ZIP Installation** | Edit `sonar.properties` | Focus of this guide |
| **Docker** | Environment variables | `sonar.jdbc.url` → `SONAR_JDBC_URL` |
| **Kubernetes** | `sonar.properties` section | In deployment config (`values.yml`) |

📚 **More info**: [System properties configuration methods](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/system-properties/configuration-methods/)

> **💡 Pro Tips:**
> - UI configuration can be automated via SonarQube Server API
> - Data Center Edition needs additional config beyond this guide
> - Get your Server ID early and request a license if DB connection won't change

---

## ✅ Before First Startup

> **🎯 Goal**: Configure database connection so SonarQube can start

### Checklist: Required sonar.properties Configuration

☐ **Configure database connection** (choose authentication method):

#### Option A: SQL User Authentication
```properties
sonar.jdbc.url=jdbc:postgresql://db-server:5432/sonarqube
sonar.jdbc.username=your_username
sonar.jdbc.password=your_password
```

#### Option B: Integrated Authentication
```properties
sonar.jdbc.url=jdbc:sqlserver://db-server:1433;databaseName=sonarqube;integratedSecurity=true
# No username/password needed
```

### Database Connection Parameters Reference

| Parameter | Purpose | Required |
|-----------|---------|----------|
| `sonar.jdbc.url` | Database connection string (server, port, database name) | ✅ Always |
| `sonar.jdbc.username` | Database user for authentication | ⚪ SQL auth only |
| `sonar.jdbc.password` | Database password for authentication | ⚪ SQL auth only |

### Post-Startup Verification

☐ **Verify startup**: SonarQube should start without database connection errors  
☐ **Access UI**: Navigate to `http://localhost:9000` (or use VM DNS name or IP for browser access)  
☐ **First login**: Use `admin`/`admin` and change password immediately

> **💡 Next Step**: Get your Server ID and [request a license](https://docs.sonarsource.com/sonarqube-server/latest/instance-administration/license-administration/#requesting-license) if database connection is stable

---

## ⚙️ After First Startup

> **🎯 Goal**: Fine-tune your SonarQube installation for production use  
> **⚠️ Important**: All changes require server restart

### Configuration Checklist

#### ☐ Reverse Proxy Setup
```properties
# Configure access log pattern for correct IP logging
sonar.web.accessLogs.pattern=%i{X-Forwarded-For} %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
```

#### ☐ Performance Tuning (Enterprise Instances)

| Parameter | Purpose |
|-----------|---------|
| `sonar.web.javaOpts` | Web server JVM options |
| `sonar.ce.javaOpts` | Compute engine JVM options |
| `sonar.search.javaOpts` | Search process JVM options |

**Example**: Increase compute engine heap for 2 workers (increase `Xmx` from default `2G` to `4G`):
```properties
sonar.ce.javaOpts=-Xmx4G -Xms128m -XX:+HeapDumpOnOutOfMemoryError
```

#### ☐ HTTP Proxy Configuration (if needed)

| Parameter | Purpose |
|-----------|---------|
| `http.proxyHost` | HTTP proxy hostname |
| `http.proxyPort` | HTTP proxy port |
| `https.proxyHost` | HTTPS proxy hostname |
| `https.proxyPort` | HTTPS proxy port |
| `http.auth.ntlm.domain` | NTLM domain |
| `http.proxyUser` | Proxy username |
| `http.proxyPassword` | Proxy password |
| `http.nonProxyHosts` | Hosts to bypass proxy |

#### ☐ LDAP Authentication Setup

If LDAP authentication is to be configured, set the security realm and configure LDAP parameters:

```properties
# Enable LDAP authentication
sonar.security.realm=LDAP

# Configure LDAP parameters (see documentation for details)
ldap.url=ldap://your-ldap-server:389
# ... additional ldap.* parameters
```

📚 **Reference**: [LDAP documentation](https://docs.sonarsource.com/sonarqube-server/latest/instance-administration/authentication/ldap/)

---

## 🖥️ UI Configuration

> **🎯 Goal**: Configure SonarQube through the web interface  
> **🔐 Required**: Global administrator privileges  
> **💡 Tip**: All UI fields are searchable

### Essential Configuration Checklist

#### ☐ General Settings → General
**Path**: Administration > Configuration > General Settings > General

| Setting | Parameter | Action |
|---------|-----------|---------|
| Server base URL | `sonar.core.serverBaseURL` | Set your server's URL |
| Default main branch name | `sonar.projectCreation.mainBranchName` | Change if not using `main` |
| Inherited rules | "Enable deactivation of inherited rules" (`sonar.qualityProfiles.allowDisableInheritedRules`) | **Disable** for safety |

#### ☐ General Settings → Security  
**Path**: Administration > Configuration > General Settings > Security

| Setting | Action | Recommendation |
|---------|--------|----------------|
| Token lifetime | Configure "Maximum allowed lifetime" | Prevent never-expiring tokens (optional) |
| Project permissions | "Enable permission management for project administrators" | **Disable** for tight control (optional) |
| Force authentication | "Force user authentication" | **Never disable** |

#### ☐ General Settings → New Code
**Path**: Administration > Configuration > General Settings > New Code

**Set default New Code**: Configure to "Number of days" → **30 days**

#### ☐ General Settings → Advanced Security
**Path**: Administration > Configuration > General Settings > Advanced Security

**Enable Advanced Security features** (if included in your license)

#### ☐ Projects → Management
**Path**: Administration > Projects > Management

**Set default project visibility**: Configure to **"Private"** ⚠️ (security best practice)

#### ☐ Projects → Background Tasks
**Path**: Administration > Projects > Background Tasks

**Configure compute engine workers**: If additional computing resources were provisioned and the JVM parameters configured appropriately, adjust the number of compute engine workers for parallel background task (analysis) processing on the SonarQube Server.

#### ☐ General Settings → AI Code Fix
**Path**: Administration > Configuration > General Settings > AI Code Fix

**Review [Terms and Conditions](https://www.sonarsource.com/legal/ai-codefix-terms/)** first  
**Enable AI Code Fix** if accepted

### Integration Configuration Checklist

#### ☐ Email Notifications
**Path**: Administration > Configuration > General Settings > Email Notification

Configure SMTP settings for email notifications

#### ☐ Authentication Providers
**Path**: Administration > Configuration > General Settings > Authentication

Configure external authentication providers as needed. Multiple providers can be enabled simultaneously.

| Provider | Purpose | Notes |
|----------|---------|-------|
| SAML | Single Sign-On authentication | Enterprise SSO integration |
| GitHub | GitHub OAuth authentication | **Recommended**: Offers additional convenient features |
| GitLab | GitLab OAuth authentication | **Recommended**: Offers additional convenient features |
| Bitbucket | Bitbucket OAuth authentication | Basic OAuth integration |

> **💡 Tip**: GitHub and GitLab authentication providers offer more convenient features beyond basic OAuth, making them preferred choices when available and when one of them can be the only authentication method to rely on (i.e., all SonarQube Server are on GitHub/GitLab).

#### ☐ DevOps Platform Integrations
**Path**: Administration > Configuration > General Settings > DevOps Platform Integrations

Configure platform integrations as needed. Multiple platforms can be integrated simultaneously to support diverse development environments.

| Platform | Features |
|----------|----------|
| GitHub | PR decoration + repository onboarding |
| GitLab | MR decoration + repository onboarding |
| Bitbucket | PR decoration + repository onboarding (Server & Cloud) |
| Azure DevOps | PR decoration + repository onboarding |

---

## 🚀 Next Steps Checklist

After completing the basic configuration:

☐ **Permission and Access Management**  
   - Configure user permissions and access controls

☐ **Project Onboarding and Analysis**  
   - Set up projects for analysis  
   - Onboard development teams
