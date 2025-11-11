# SonarQube Server Guides

This repository provides a collection of guides and practical examples designed to complement the official SonarQube Server documentation. While the official documentation covers installation, configuration, and core concepts, this repository focuses on real-world scenarios, and advanced usage patterns that can help you get the most out of your SonarQube Server deployment.

## Table of Contents

- Configuration
  - [SonarQube Server configuration cheat sheet](configuration/configuration-cheat-sheet.md)
- [Helm Chart Guides](helm-chart/)
  - [Common Guides](helm-chart/common/) - General guidelines applicable to all SonarQube deployments
    - [When to deploy SonarQube Server on Kubernetes (and when not)](helm-chart/common/when-to-deploy-on-k8s.md)
    - [Connect SonarQube to an external database](helm-chart/common/connect-external-db.md)
    - [Download SonarQube images from a private repository](helm-chart/common/images-from-private-repo.md)
    - [Changing the admin's password from the Helm chart](helm-chart/common/change-admin-password.md)
  - [SonarQube EE/DE/CB Chart](helm-chart/ee-de-cb/) - Guides specific to the `sonarqube` Helm chart
    - [Pod assignment to Kubernetes nodes](helm-chart/ee-de-cb/pod-to-node-assignment.md)
  - [SonarQube DCE Chart](helm-chart/dce/) - Guides specific to the `sonarqube-dce` Helm chart
    - [Secure the communication within the cluster](helm-chart/dce/secure-communication.md)
    - [Pod assignment to Kubernetes nodes](helm-chart/dce/pod-to-node-assignment.md)
    - [Download plugins using self-signed certificate and basic authentication](helm-chart/dce/download-plugins-self-signed-cert.md)
- [Database](database/) - Database-related operations and tools
  - [Database vendor migration plan](database/db-copy-plan.md)
  - [Running the SonarQube Server DB Copy Tool on Kubernetes](database/db-copy-tool-on-kubernetes.md)
- [Examples](examples/) - Practical examples and reference implementations
  - [Kubernetes Examples](examples/kubernetes/) - Kubernetes manifests and configurations

## SonarQube Server Scripts

For helpful scripts related to SonarQube Server administrative tasks, check out the sister repository: [sonarqube-server-scripts](https://github.com/lukas-frystak-sonarsource/sonarqube-server-scripts).

