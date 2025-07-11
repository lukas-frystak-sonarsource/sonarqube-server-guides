# Helm chart-related guides

This section provides a collection of guides related to the installation of SonarQube Server on Kubernetes using the Sonar-provided Helm chart.

the guides and examples are structured in the following way:
- Common
- `sonarqube` chart specific
- `sonarqube-dce` chart specific

Table of contents:
- Common
  - [When to deploy SQS on Kubernetes (and when not)](common/when-to-deploy-on-k8s.md)
  - [Connect SonarQube to an external database](common/connect-external-db.md)
  - [Download SonarQube images from a private repository](common/images-from-private-repo.md)
  - [Changing the admin's password from the Helm chart](common/change-admin-password.md)
- `sonarqube` chart
- `sonarqube-dce` chart
  - [Secure the communication within the cluster](dce/secure-communication.md)