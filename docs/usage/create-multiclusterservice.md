# Deploy beach-head services using MultiClusterService

The `MultiClusterService` object is used to deploy beach-head services on multiple matching clusters.

## Creation

The `MultiClusterService` object can be created with the following YAML:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: MultiClusterService
metadata:
  name: <name>
spec:
  clusterSelector:
    matchLabels:
      <key1>: <value1>
      <key2>: <value2>
      . . .
  services:
  - template: <servicetemplate-1-name>
    name: <release-name>
    namespace: <release-namespace>
  servicesPriority: 100
  stopOnConflict: false
```

## Matching Multiple Clusters

Consider the following example where 2 clusters have been deployed using ManagedCluster objects:
```sh
➜  ~ kubectl get managedclusters.hmc.mirantis.com -n hmc-system
NAME             READY   STATUS
dev-cluster-1   True    ManagedCluster is ready
dev-cluster-2   True    ManagedCluster is ready
➜  ~ 
➜  ~ 
➜  ~  kubectl get cluster -n hmc-system --show-labels
NAME           CLUSTERCLASS     PHASE         AGE     VERSION   LABELS
dev-cluster-1                  Provisioned   2h41m             app.kubernetes.io/managed-by=Helm,helm.toolkit.fluxcd.io/name=dev-cluster-1,helm.toolkit.fluxcd.io/namespace=hmc-system,sveltos-agent=present
dev-cluster-2                  Provisioned   3h10m             app.kubernetes.io/managed-by=Helm,helm.toolkit.fluxcd.io/name=dev-cluster-2,helm.toolkit.fluxcd.io/namespace=hmc-system,sveltos-agent=present
```

> EXAMPLE: 
> Spec for `dev-cluster-1` ManagedCluster (only sections relevant to beach-head services):
> ```yaml
> apiVersion: hmc.mirantis.com/v1alpha1
> kind: ManagedCluster
> metadata:
>   . . . 
>   name: dev-cluster-1
>   namespace: hmc-system
> spec:
>   . . .
>   services:
>   - name: kyverno
>     namespace: kyverno
>     template: kyverno-3-2-6
>   - name: ingress-nginx
>     namespace: ingress-nginx
>     template: ingress-nginx-4-11-0
>   servicesPriority: 100
>   stopOnConflict: false
>   . . .
> ```
> 
> Spec for `dev-cluster-2` ManagedCluster (only sections relevant to beach-head services):
> ```yaml
> apiVersion: hmc.mirantis.com/v1alpha1
> kind: ManagedCluster
> metadata:
>   . . .
>   name: dev-cluster-2
>   namespace: hmc-system
> spec:
>   . . .
>   services:
>   - name: ingress-nginx
>     namespace: ingress-nginx
>     template: ingress-nginx-4-11-0
>   servicesPriority: 500
>   stopOnConflict: false
>   . . .
> ```

> NOTE: See [Deploy beach-head Services using Managed Cluster](deploy-services-managedcluster.md) for how to use beach-head services with ManagedCluster.

Now the following `global-ingress` MultiClusterService object is created with the following spec:

```yaml
apiVersion: hmc.mirantis.com/v1alpha1
kind: MultiClusterService
metadata:
  name: global-ingress
spec:
  clusterSelector:
    matchLabels:
      app.kubernetes.io/managed-by: Helm
  services:
  - name: ingress-nginx
    namespace: ingress-nginx
    template: ingress-nginx-4-11-3
  servicesPriority: 300
  stopOnConflict: false
```


This MultiClusterService will match any CAPI cluster with the label `app.kubernetes.io/managed-by: Helm` and deploy
version 4.11.3 of ingress-nginx service on it.

### Configuring Custom Values

Refer to "Configuring Custom Values" in [Deploy beach-head Services using Managed Cluster](./deploy-services-managedcluster.md).

### Templating Custom Values

Refer to "Templating Custom Values" in [Deploy beach-head Services using Managed Cluster](./deploy-services-managedcluster.md).

### Services Priority and Conflict

The `.spec.servicesPriority` field is used to specify the priority for the services managed by a ManagedCluster or MultiClusterService object.
Considering the example above:

1. ManagedCluster `dev-cluster-1` manages deployment of kyverno (v3.2.6) and ingress-nginx (v4.11.0) with `servicesPriority=100` on its cluster.
2. ManagedCluster `dev-cluster-2` manages deployment of ingress-nginx (v4.11.0) with `servicesPriority=500` on its cluster.
3. MultiClusterService `global-ingress` manages deployment of ingress-nginx (v4.11.3) with `servicesPriority=300` on both clusters.

This scenario presents a conflict on both the clusters as the MultiClusterService is attempting to deploy v4.11.3 of ingress-nginx
on both whereas the ManagedCluster for each is attempting to deploy v4.11.0 of ingress-nginx.

This is where `.spec.servicesPriority` can be used to specify who gets the priority. Higher number means higer priority and vice versa. In this example:
1. ManagedCluster `dev-cluster-1` would lose and its ingress-nginx (v4.11.3) would be deployed on its cluster.
2. ManagedCluster `dev-cluster-2` would win and ingress-nginx (v4.11.0) would be deployed on its cluster.

> NOTE: If servicesPriority are equal, the first one to reach the cluster wins and deploys its beach-head services.

## Checking Status

The status for the MultiClusterService object will show the deployment status for the beach-head services managed
by it on each of the CAPI target clusters that it matches. Consider the same example where 2 ManagedClusters
and 1 MultiClusterService is deployed.

> EXAMPLE: Status for `global-ingress` MultiClusterService
> ```yaml
> apiVersion: hmc.mirantis.com/v1alpha1
> kind: MultiClusterService
> metadata:
>   . . .
>   name: global-ingress
>   resourceVersion: "38146"
>   . . .
> spec:
>   clusterSelector:
>     matchLabels:
>       app.kubernetes.io/managed-by: Helm
>   services:
>   - name: ingress-nginx
>     namespace: ingress-nginx
>     template: ingress-nginx-4-11-3
>   servicesPriority: 300
>   stopOnConflict: false
> status:
>   conditions:
>   - lastTransitionTime: "2024-10-25T08:36:24Z"
>     message: ""
>     reason: Succeeded
>     status: "True"
>     type: SveltosClusterProfileReady
>   - lastTransitionTime: "2024-10-25T08:36:24Z"
>     message: MultiClusterService is ready
>     reason: Succeeded
>     status: "True"
>     type: Ready
>   observedGeneration: 1
>   services:
>   - clusterName: dev-cluster-2
>     clusterNamespace: hmc-system
>     conditions:
>     - lastTransitionTime: "2024-10-25T08:36:35Z"
>       message: |
>         cannot manage chart ingress-nginx/ingress-nginx. ClusterSummary p--dev-cluster-2-capi-dev-cluster-2 managing it.
>       reason: Failed
>       status: "False"
>       type: Helm
>     - lastTransitionTime: "2024-10-25T08:36:25Z"
>       message: 'Release ingress-nginx/ingress-nginx: ClusterSummary p--dev-cluster-2-capi-dev-cluster-2
>         managing it'
>       reason: Conflict
>       status: "False"
>       type: ingress-nginx.ingress-nginx/SveltosHelmReleaseReady
>   - clusterName: dev-cluster-1
>     clusterNamespace: hmc-system
>     conditions:
>     - lastTransitionTime: "2024-10-25T08:36:24Z"
>       message: ""
>       reason: Provisioned
>       status: "True"
>       type: Helm
>     - lastTransitionTime: "2024-10-25T08:36:25Z"
>       message: Release ingress-nginx/ingress-nginx
>       reason: Managing
>       status: "True"
>       type: ingress-nginx.ingress-nginx/SveltosHelmReleaseReady
> ```

The status under `.status.services` shows a conflict for `dev-cluster-2` as expected because the MultiClusterService has a lower priority.
Whereas, it shows provisioned for `dev-cluster-1` because the MultiClusterService has a higher priority.

> EXAMPLE: Status for `dev-cluster-1` ManagedCluster (only sections relevant to beach-head services):
> ```yaml
> apiVersion: hmc.mirantis.com/v1alpha1
> kind: ManagedCluster
> metadata:
>   . . . 
>   name: dev-cluster-1
>   namespace: hmc-system
>   . . .
> spec:
>   . . .
>   services:
>   - name: kyverno
>     namespace: kyverno
>     template: kyverno-3-2-6
>   - name: ingress-nginx
>     namespace: ingress-nginx
>     template: ingress-nginx-4-11-0
>   servicesPriority: 100
>   stopOnConflict: false
>   . . .
> status:
>   . . .
>   services:
>   - clusterName: dev-cluster-1
>     clusterNamespace: hmc-system
>     conditions:
>     - lastTransitionTime: "2024-10-25T08:36:35Z"
>       message: |
>         cannot manage chart ingress-nginx/ingress-nginx. ClusterSummary global-ingress-capi-dev-cluster-1 managing it.
>       reason: Provisioning
>       status: "False"
>       type: Helm
>     - lastTransitionTime: "2024-10-25T07:44:43Z"
>       message: Release kyverno/kyverno
>       reason: Managing
>       status: "True"
>       type: kyverno.kyverno/SveltosHelmReleaseReady
>     - lastTransitionTime: "2024-10-25T08:36:25Z"
>       message: 'Release ingress-nginx/ingress-nginx: ClusterSummary global-ingress-capi-dev-cluster-1
>         managing it'
>       reason: Conflict
>       status: "False"
>       type: ingress-nginx.ingress-nginx/SveltosHelmReleaseReady
> ```

The status under `.status.services` for ManagedCluster `dev-cluster-1` shows that it is managing kyverno but unable to manage ingress-nginx because
another object with higher priority is managing it, so it shows a conflict instead.

> EXAMPLE: Status for `dev-cluster-2` ManagedCluster (only sections relevant to beach-head services):
> ```yaml
> apiVersion: hmc.mirantis.com/v1alpha1
> kind: ManagedCluster
> metadata:
>   . . .
>   name: dev-cluster-2
>   namespace: hmc-system
>   resourceVersion: "30889"
>   . . .
> spec:
>   . . .
>   services:
>   - name: ingress-nginx
>     namespace: ingress-nginx
>     template: ingress-nginx-4-11-0
>   servicesPriority: 500
>   stopOnConflict: false
>   . . .
> status:
>   . . .
>   services:
>   - clusterName: dev-cluster-2
>     clusterNamespace: hmc-system
>     conditions:
>     - lastTransitionTime: "2024-10-25T08:18:22Z"
>       message: ""
>       reason: Provisioned
>       status: "True"
>       type: Helm
>     - lastTransitionTime: "2024-10-25T08:18:22Z"
>       message: Release ingress-nginx/ingress-nginx
>       reason: Managing
>       status: "True"
>       type: ingress-nginx.ingress-nginx/SveltosHelmReleaseReady
> ```

The status under `.status.services` for ManagedCluster `dev-cluster-2` shows that it is managing ingress-nginx as expected since it has a higher priority.

## Parameter List

Refer to "Parameter List" in [Deploy beach-head Services using Managed Cluster](./deploy-services-managedcluster.md).