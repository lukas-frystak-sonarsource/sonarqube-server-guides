# Secure the communication within the cluster

This page describes how to enable and configure secure communication on SonarQube Server Data Center Edition deployed on Kubernetes installed with the [SonarQube Helm chart](https://artifacthub.io/packages/helm/sonarqube/sonarqube-dce).

In this context, secure communication means encrypting the communication between the application and search nodes. Once the feature is enabled, the ElasticSearch cluster is accessed via HTTPS as opposed to the default HTTP access.

While the configuration is documented in the SonarQube Server helm chart repository [here](https://github.com/SonarSource/helm-chart-sonarqube/tree/master/charts/sonarqube-dce#secure-the-communication-within-the-cluster), this page aims to provide more structured steps and a complete example.

> [!NOTE]
> As with many things related to Kubernetes, there are many different ways of doing everything. This page provides complete instructions, but should still be treated as an example. The steps needs to be adapted to the specific environment where the steps are being applied.

> [!TIP]
> The example commands in this document are for PowerShell on Windows. It is not guaranteed that they will work on other operating systems. The reader needs to adapt the commands for different operating systems and shells if required.

**The configuration of the secure communication happens in two parts:**
1. Generate the required TLS certificate for ElasticSearch encryption using `elasticsearch-certutil`
1. Configure the  SonarQube Server helm chart

## Generate the required TLS certificate

### Get search pod hostnames

The hostnames of all ElasticSearch nodes and the corresponding service must be included in the certificate file that will be generated in the following step. The hostnames are used as input for the certificate generation.

- If SonarQube is already running on Kubernetes cluster, you can get the search containers' host names by executing `hostname -f` in each of them. Additionally, it is needed to get the service name. An an example, the following PowerShell command gets the required values:
  ```
  $sqNamespace = "sonarqube"; `
  $(kubectl get pods -n $sqNamespace -o json | ConvertFrom-Json).items.metadata.name `
  | Select-String "search" `
  | Foreach-Object { kubectl exec -it $_ -n $sqNamespace -- sh -c "hostname -f" }; `
  $((kubectl get service -n $sqNamespace -o json | ConvertFrom-Json).items.metadata.name | Select-String "search$").Line;
  ```
- If it's not possible to execute the commands, the hostnames have the following format:
  ```
  Pods:    <release-name>-<chart-name>-<pod>.<release>-<chart-name>-search.<namespace>.svc.cluster.local
  Service: <release-name>-<chart-name>-search
  ```
- Here are example values that represent the output from the above PowerShell snippet and that also correspond to the pattern above with these parameters:
  - release => `sonarqube-rel`
  - chart name => `sonarqube-dce`
  - namespace => `sonarqube`
  ```
  sonarqube-rel-sonarqube-dce-search-0.sonarqube-rel-sonarqube-dce-search.sonarqube.svc.cluster.local
  sonarqube-rel-sonarqube-dce-search-1.sonarqube-rel-sonarqube-dce-search.sonarqube.svc.cluster.local
  sonarqube-rel-sonarqube-dce-search-2.sonarqube-rel-sonarqube-dce-search.sonarqube.svc.cluster.local
  sonarqube-rel-sonarqube-dce-search
  ```

### Generate the required certificate

A TLS certificate is needed to encrypt the cluster traffic and this section describes how to generate it. This is done in two steps:
1. Generate a Certificate Authority (CA) (based on ElasticSearch documenation [here](https://www.elastic.co/docs/deploy-manage/security/set-up-basic-security#generate-certificates))
1. Generate a TLS certificate for ElasticSearch nodes (based on ElasticSearch documenation [here](https://www.elastic.co/docs/deploy-manage/security/set-up-basic-security-plus-https#encrypt-http-communication))

Different tools may be used, but it is recommended to use the `elasticsearch-certutil` as it corresponds to ElasticSearch documentation.

The cert tool ships with ElasticSearch so the easiest way to use it is to download the ElasticSearch binaries. The tool can then be found in the `bin` directory.

Below are example commands for generating the CA and certificate using the tool from ElasticSearch binaries.
- Generate a Certificate Authority (CA)
  ```
  C:\elasticsearch-8.16.1\bin\elasticsearch-certutil.bat ca --out "elastic-stack-cert-auth.p12"
  ```
  - During the execution of the commnad, you will be prompted to enter a password for the CA. The password will be used during the next step.
  - The file is saved in the ElasticSearch install directory.
- Generate a TLS certificate for ElasticSearch nodes
  ```
  C:\elasticsearch-8.16.1\bin\elasticsearch-certutil.bat http
  ```
  The generation process with the `http` command is interactive. Follow the instructions you receive on the command line...
  - Generate a CSR? --> `N`
  - Use an existing CA? --> `y`
  - CA Path --> `C:/elasticsearch-8.16.1/elastic-stack-cert-auth.p12`
  - All above hostnames must be included in a single certificate file.
  - Password for `elastic-stack-cert-auth.p12` --> input the value set when generating the CA
  - For how long should your certificate be valid? --> `5y` (or enter any other desired validity, e.g., 90 days - `90d`)
  - Generate a certificate per node? --> `N`
  - Enter all the hostnames that you need, one per line. --> use the output hostnames + service name as explained above
  - Is this correct? --> `Y`
  - Enter IP addresses --> skip by pressing ENTER
  - Is this correct? --> `Y`
  - Do you wish to change any of these options? --> `N`
  - Enter the password for the `http.p12` file (twice) --> choose a password. *This value is needed in the helm chart.*
  - What filename should be used for the outpit zip file? --> choose the desired path. E.g., default

The `http.p12` is in the generated zip file: `<zip>/elasticsearch/http.p12`. The file has to be extracted and renamed to `elastic-stack-ca.p12` (as the file name is currently hardcoded in the SQS helm chart).

## Configure the Helm chart

The secure communication between SonarQube application and search node is enabled by configuring the below parameters.

- Create a secret from the certificate file. For example:
  ```
  kubectl create secret generic --from-file "elastic-stack-ca.p12" -n sonarqube es-cert-sonarqube
  ```
- Set `searchNodes.searchAuthentication.enabled` to `true`
- Set `searchNodes.searchAuthentication.keyStoreSecret` to the name of the secret created in the previous step (`es-cert-sonarqube` in this example)
- Set `searchNodes.searchAuthentication.keyStorePassword` to the password used during certificate generation
  - Alternatively, the password can be set from a secret: `searchNodes.searchAuthentication.keyStorePasswordSecret`
- Set `searchNodes.searchAuthentication.userPassword` to any value
- Set `nodeEncryption.enabled` to `true`

### Verify the secure communication is enabled

If the configuration is correct, the cluster will start up without errors. If it does start, the following log messsages confirm the secure communication was enabled.

- Search pod logs
  ```
  2025.06.18 05:41:04 INFO  sonarqube-rel-sonarqube-dce-search-0 es[][o.e.x.s.Security] Security is enabled
  ```
- App pod logs
  ```
  2025.06.17 10:22:48 INFO  sonarqube-rel-sonarqube-dce-app-77988d8787-4hxzb web[][o.s.s.e.EsClientProvider] Connected to remote Elasticsearch: [https://sonarqube-rel-sonarqube-dce-search:9001]
  ```
  Note: the HTTPS in the ES URL.