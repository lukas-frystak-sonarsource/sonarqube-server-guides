# Changing the admin's password from the Helm chart

The SonarQube Server Helm chart allows you to securely set a new password for the default local `admin` account during deployment or upgrade. This ensures your instance is never exposed with the default credentials, eliminating the need for manual password changes after startup.

### When to use this feature

Configure the admin password during the initial installation or any subsequent upgrade to prevent exposure of the default credentials at any stage of your deployment.

**Note:** This feature requires both the *current* and *new* admin passwords to be provided, as it changes the password from the existing value to a new one. It is intended for one-time password updates rather than for setting a new password on every deployment.

### Configuration options

The admin password can be set using multiple approaches:
- Directly in the Helm chart's values file
- Via Kubernetes secrets (recommended)

This flexibility allows you to choose the most appropriate method based on your requirements and deployment practices.

## Defining the password in the values file

The password change can be defined in the values file in plain text using the two parameters below:
 - `setAdminPassword.newPassword`
 - `setAdminPassword.currentPassword`

```yaml
setAdminPassword:
  newPassword: <enter_your_password>
  currentPassword: admin
```

## Defining the password via a secret

It is preferable to define the password in a secret rather than in a plain text in the values file. In this case, the `setAdminPassword.passwordSecretName` parameter should be set instead of the two above parameters.

```
kubectl create secret generic admin-password-secret-name --from-literal=password=adminNewPassword --from-literal=currentPassword=admin -n <sonarqube-namespace>
```
Then you can set the *password secret name* in the vales file:
```
setAdminPassword:
    passwordSecretName: admin-password-secret-name
```
Or in the heml upgrade/install command:
```
helm upgrade --install ... --set setAdminPassword.passwordSecretName=admin-password-secret-name ...
```

> [!TIP]
> Keep in mind the new password must meet SonarQube Server password complexity requirements:
> - 12 characters
> - 1 upper case letter
> - 1 lower case letter
> - 1 number
> - 1 special character