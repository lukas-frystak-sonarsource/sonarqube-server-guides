# Pod assignment to Kubernetes nodes (EE/DE/CB)

When deploying SonarQube Server pods, it's sometimes necessary to ensure they are scheduled on specific Kubernetes worker nodes. This is often due to particular resource or configuration requirements. For instance, the SonarQube search pods, which run Elasticsearch, have specific [Linux host requirements](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/pre-installation/linux/#configuring-the-maximum-number-of-open-files-and-other-limits). To accommodate this, you might provision a dedicated node pool. The SonarQube Server Helm chart configuration can then be used to ensure that the pods—whether it's just the search pods, the application pods, or all of them—are deployed to these specific worker nodes.

When it comes to [assigning pods to nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector), the SonarQube Server Helm chart implements three mechanisms:
- [Node labels + node selector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#built-in-node-labels)
  - The simplest recommended form of node selection constraint. You specify node labels in your Pod specification, and Kubernetes schedules the Pod only onto nodes that have those labels.
- [Affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
  - Node affinity is a property of Pods that *attracts* them to a set of nodes (either as a preference or a hard requirement).
  - Affinity provides more control over the selection logic but increases complexity.
- [Taints and tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
  - *Taints* are the opposite of *affinity* - they allow a node to repel a set of pods.

For more detail, refer to the official Kubernetes documentation: [Scheduling, Preemption and Eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/).

> [!WARNING]
> While affinity and taints/tolerations are supported, use these features cautiously. Complex affinity rules can cause rescheduling and disruptions for stateful pods like those of SonarQube Server.

### SonarQube Server Helm chart options/configuration
- The following options are supported in the Helm chart:
  - `nodeSelector`
  - `affinity`
  - `tolerations`

#### Example: Assign SonarQube Server pod to a dedicated node using `nodeSelector`

Label the required node(s). For example, this can be done with `kubectl` like this:
```sh
kubectl label node <node> sonarqube=true
```

Set the `nodeSelector` field in the Helm chart:
```yaml
nodeSelector:
  sonarqube: "true"
```