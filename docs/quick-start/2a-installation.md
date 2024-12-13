## Requirements

Project 2A requires a Kubernetes cluster. It can be of any type and will become
the 2A _management cluster_.

If you don't have a Kubernetes cluster yet, consider using
[k0s](https://docs.k0sproject.io/stable/install/).

The following instructions assume:

- Your `kubeconfig` points to the correct Kubernetes cluster.
- You have [Helm](https://helm.sh/docs/intro/install/) installed.
- You have [kubectl](https://kubernetes.io/docs/tasks/tools/) installed.

### Helpful Tools

It may be helpful to have the following tools installed:

- [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html?highlight=clusterctl#install-clusterctl)
- [Mirantis Lens](https://k8slens.dev/)
- [k9s](https://k9scli.io/)

## Installation via Helm

```bash
helm install hmc oci://ghcr.io/mirantis/hmc/charts/hmc --version 0.0.4 -n hmc-system --create-namespace
```

## Verification

The installation takes a few minutes until 2A and its subcomponents are
fully installed and configured.

### Verify Core Components are running

Check pods are running in the `hmc-system` namespace with the following command:

```bash
kubectl get pods -n hmc-system
```

The output should be similar to:
```bash
NAME                                                           READY   STATUS
azureserviceoperator-controller-manager-86d566cdbc-rqkt9       1/1     Running
capa-controller-manager-7cd699df45-28hth                       1/1     Running
capi-controller-manager-6bc5fc5f88-hd8pv                       1/1     Running
capv-controller-manager-bb5ff9bd5-7dsr9                        1/1     Running
capz-controller-manager-5dd988768-qjdbl                        1/1     Running
helm-controller-76f675f6b7-4d47l                               1/1     Running
hmc-cert-manager-7c8bd964b4-nhxnq                              1/1     Running
hmc-cert-manager-cainjector-56476c46f9-xvqhh                   1/1     Running
hmc-cert-manager-webhook-69d7fccf68-s46w8                      1/1     Running
hmc-cluster-api-operator-79459d8575-2s9jc                      1/1     Running
hmc-controller-manager-64869d9f9d-zktgw                        1/1     Running
k0smotron-controller-manager-bootstrap-6c5f6c7884-d2fqs        2/2     Running
k0smotron-controller-manager-control-plane-857b8bffd4-zxkx2    2/2     Running
k0smotron-controller-manager-infrastructure-7f77f55675-tv8vb   2/2     Running
source-controller-5f648d6f5d-7mhz5                             1/1     Running
```

Checking pods are running in the `projectsveltos` namespace with the following command:
```sh
kubectl get pods -n projectsveltos
```

The output should be similar to:
```sh
NAME                                     READY   STATUS    RESTARTS   AGE
access-manager-cd49cffc9-c4q97           1/1     Running   0          16m
addon-controller-64c7f69796-whw25        1/1     Running   0          16m
classifier-manager-574c9d794d-j8852      1/1     Running   0          16m
conversion-webhook-5d78b6c648-p6pxd      1/1     Running   0          16m
event-manager-6df545b4d7-mbjh5           1/1     Running   0          16m
hc-manager-7b749c57d-5phkb               1/1     Running   0          16m
sc-manager-f5797c4f8-ptmvh               1/1     Running   0          16m
shard-controller-767975966-v5qqn         1/1     Running   0          16m
sveltos-agent-manager-56bbf5fb94-9lskd   1/1     Running   0          15m
```

If you have fewer pods, give 2A more time to reconcile all the pods.

### Verify 2A templates have been successfully reconciled

For additional verification, check that the example templates packaged with 2A have
been installed and are valid.

Check `ProviderTemplate` objects with:

```sh
kubectl get providertemplate -n hmc-system
```

The output should be similar to:

```sh
NAME                                 VALID
cluster-api-0-0-3                    true
cluster-api-provider-aws-0-0-3       true
cluster-api-provider-azure-0-0-3     true
cluster-api-provider-vsphere-0-0-3   true
hmc-0-0-4                            true
k0smotron-0-0-3                      true
projectsveltos-0-42-0                true
```

Check `ClusterTemplate` objects with:

```bash
kubectl get clustertemplate -n hmc-system
```

The output should be similar to:

```bash
NAME                                VALID
aws-eks-0-0-2                       true
aws-hosted-cp-0-0-3                 true
aws-standalone-cp-0-0-3             true
azure-hosted-cp-0-0-3               true
azure-standalone-cp-0-0-3           true
vsphere-hosted-cp-0-0-3             true
vsphere-standalone-cp-0-0-3         true
```

Check `ServiceTemplate` objects with:

```sh
kubectl get servicetemplate -n hmc-system
```

The output should be similar to:

```sh
NAME                   VALID
ingress-nginx-4-11-0   true
ingress-nginx-4-11-3   true
kyverno-3-2-6          true
```

### Next Step

Now you can configure your Infrastructure Provider of choice and create your
first Managed Cluster.

Jump to any of the following Infrastructure Providers for specific instructions:

- [AWS Quick Start](aws.md)
- [Azure Quick Start](azure.md)
- [vSphere Quick Start](vsphere.md)
