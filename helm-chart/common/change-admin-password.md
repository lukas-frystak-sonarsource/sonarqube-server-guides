# Changing the admin's password from the Helm chart

The SonarQube Server Helm chart provides the ability to change the password of the default local `admin` account. This feature eliminates the need for administrators to manually log in and change the password after startup.

### When to use this feature

This password configuration can be applied during the first SonarQube Server startup or at any point later during normal operation. It is particularly valuable during the initial installation to ensure the instance is never exposed with the well-known default credentials.

### Configuration options

The admin password can be set using multiple approaches:

- Directly in the Helm chart's values file
- Via Kubernetes secrets (recommended)

This flexibility allows you to choose the most appropriate method based on your requirements and deployment practices.

## Defining the password in the values file
`setAdminPassword.newPassword`
`setAdminPassword.currentPassword`
`setAdminPassword.passwordSecretName`
```
kubectl create secret generic admin-password-secret-name --from-literal=password=adminNewPassword --from-literal=currentPassword=admin -n <sonarqube-namespace>
```