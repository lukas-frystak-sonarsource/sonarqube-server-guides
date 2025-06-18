# Download SonarQube images from a private repository

When the Helm chart is installed on a Kubernetes cluster, the SonarQube image(s) must be downloaded (pulled) from an accessible image repository.
By default, Docker images are pulled from Docker Hub. This is then how the default image is defined in the values file:
```yaml
image:
  repository: sonarqube
```

However, in some cases, the images may be stored in an **internal/private repository**. This requires the complete path to the registry and container to be set in the `repository` value. The tag should remain the same, assuming the image(s) arein the same way as the images published by Sonar (this is certainly recommended). For example, it might look like this:
```yaml
image:
    repository: ghcr.io/sonarsource-demos:sonarqube
    tag: 2025.1.0-datacenter-search
```

Or like this in the Data Center Edition (DCE) helm chart:
```yaml
searchNodes:
  image:
    repository: ghcr.io/sonarsource-demos:sonarqube
    tag: 2025.1.0-datacenter-search
```

If authentication is needed to pull images from a private repository, the credentials must be defined in a secret. For example:
```yaml
image:
  ...
  pullSecrets:
    - name: my-repo-secret
```