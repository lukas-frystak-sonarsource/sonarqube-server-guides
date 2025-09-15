# Download plugins using a self-signed certificate and basic authentication

The helm chart currently doesn't support downloading plugins while using self-signed certificates. This page documents how to implement a custom plugins download using both a self-signed certificate and basic authentication (.netrc file).

The custom implementation follows the native plugin installation feature. However, as it is a custom implementation, it will rely on `extraInitContainers`, `extraVolumes`, and `extraVolumeMounts`.

**Implementation steps:**
- Native plugins download and installation must be disabled.
- 2 secrets must be created: one from the self-signed certificate, one for basic authentication based on the `.netrc` file.
    - Certificate secret example creation:
      ```
      kubectl create secret generic artifactory-cert --from-file=crt=<cert-file> -n sonarqube
      ```
    - Authentication secret example creation:
      ```
      kubectl create secret generic artifactory-auth-secret --from-file=netrc=.netrc -n sonarqube
      ```
- Add extra volumes:
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
            path: crt
      ```
- Add extra volume mounts
    - This needs to be set in the `applicationNodes.extraVolumeMounts` property:
      ```
      extraVolumeMounts:
        - name: custom-plugin-install
          mountPath: /opt/sonarqube/extensions/plugins
      ```
- Add extra init container:
    - `applicationNodes.extraInitContainers`
      ```yaml
      extraInitContainers:
      - name: custom-plugin-install
          image: sonarqube:2025.3.1-datacenter-app
          imagePullPolicy: IfNotPresent
          command:
          - "sh"
          - "-c"
          - |
              rm -f /opt/sonarqube/extensions/plugins/*;
              cd /opt/sonarqube/extensions/plugins;
              curl --cacert /tmp/secrets/ca-certs/* --netrc-file /root/.netrc -vfsSLO "http://jfrog-rel-artifactory-nginx.jfrog.svc.cluster.local/artifactory/gen-repo/sonar-delphi-plugin-1.18.1.jar";
          volumeMounts:
          - name: custom-plugin-install
              mountPath: /opt/sonarqube/extensions/plugins
          - name: custom-plugins-netrc-file
              mountPath: /root
          - mountPath: /tmp/secrets/ca-certs
              name: ca-certs
      ```