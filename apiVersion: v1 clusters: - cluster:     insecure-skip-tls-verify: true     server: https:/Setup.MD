# Setup RBAC with MKE based k8s installtion

## Prerequisites

### This installation guide was tested with the following components:

- Mesosphere DC/OS Enterprise 1.12.3 or higher
- Mesosphere Kubernetes Engine 2.2.0-1.13.3 or higher
- Edge-LB 1.3.0 or higher 
- Access to K8s cluster running in MKE with Aurhorization mode as `RBAC` find the link below on how to do that.
  https://docs.mesosphere.com/services/kubernetes/2.3.3-1.14.3/operations/authn-and-authz/#rbac

Download and unpack Istio to your workstation:

```bash
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.8 sh -
cd istio-1.1.8
```

Changes and Updated KubeConfig:

```bash
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://54.191.244.89:6443
  name: kubernetes-cluster1
contexts:
- context:
    cluster: kubernetes-cluster1
    user: kubernetes-cluster1
  name: kubernetes-cluster1
- context:
    cluster: kubernetes-cluster1
    user: app-dev-sa
  name: app-dev-sa
current-context: kubernetes-cluster1
kind: Config
preferences: {}
users:
- name: kubernetes-cluster1
  user:
    token:  XXXXXXXXXXXXXXX
- name: app-dev-sa
  user:
    token: XXXXXXXXXXXXXXXXXXXX

```
