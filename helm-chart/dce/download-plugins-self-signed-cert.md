# Download plugins using a self-signed certificate and basic authentication

The helm chart currently doesn't support downloading plugins while using self-signed certificates. This page documents how to implement a custom plugins download using both a self-signed certificate and basic authentication (.netrc file).

The custom implementation follows the native plugin installation feature. However, as it is a custom implementation, it will rely on `extraInitContainers`, `extraVolumes`, and `extraVolumeMounts`.

**Implementation steps:**
- Native plugins download and installation must be disabled. i.e., do not set the `applicationNodes.plugins` parameter.
- Two secrets must be created: one from the self-signed certificate and one for basic authentication based on the `.netrc` file.
    - Certificate secret example creation:
      ```sh
      kubectl create secret generic artifactory-cert --from-file=crt=jfrog-tls.crt -n sonarqube
      ```
    - Authentication secret example creation:
      ```sh
      kubectl create secret generic artifactory-auth-secret --from-file=netrc=.netrc -n sonarqube
      ```
    - Note: the file names are important and used in the next steps.
- Add extra volumes:
    - Three extra volumes are added
      - An empty dir volume that will hold the downloaded plugins
      - Two volumes created from secrets: they hold the certificate and authentication files, respectively.
    - This needs to be set in the `applicationNodes.extraVolumes` property:
      ```yaml
      extraVolumes:
        - name: custom-plugin-install
          emptyDir: {}
        - name: custom-plugins-netrc-file
          secret:
            secretName: artifactory-auth-secret
            items:
            - key: netrc
              path: .netrc
        - name: custom-plugins-crt-file
          secret:
            secretName: artifactory-cert
            items:
            - key: crt
              path: jfrog-tls.crt
      ```
- Add extra volume mounts:
    - The volume holding the downloaded plugins must be mounted to the SonarQube Server application container.
    - This needs to be set in the `applicationNodes.extraVolumeMounts` property:
      ```yaml
      extraVolumeMounts:
        - name: custom-plugin-install
          mountPath: /opt/sonarqube/extensions/plugins
      ```
- Add extra init container:
  - The extra init container downloads the plugins.
  - If multiple plugins are required, the `curl` command must be copied on new lines with additional plugin URLs.
  - This needs to be set in the `applicationNodes.extraInitContainers` property:
    ```yaml
    extraInitContainers:
    - name: custom-plugin-install
      image: sonarqube:2025.1.3-datacenter-app
      imagePullPolicy: IfNotPresent
      command:
        - "sh"
        - "-c"
        - |
            rm -f /opt/sonarqube/extensions/plugins/*;
            cd /opt/sonarqube/extensions/plugins;
            curl --cacert /tmp/secrets/ca-certs/* --netrc-file /root/.netrc -fsSLO "<PLUGIN_URL>";
      volumeMounts:
        - name: custom-plugin-install
          mountPath: /opt/sonarqube/extensions/plugins
        - name: custom-plugins-netrc-file
          mountPath: /root
        - mountPath: /tmp/secrets/ca-certs
          name: custom-plugins-crt-file
    ```