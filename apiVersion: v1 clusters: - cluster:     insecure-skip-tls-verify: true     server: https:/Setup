# Setup Istio with MKE

## Installation

This installation guide was tested with the following components:

- Mesosphere DC/OS Enterprise 1.12.3
- Mesosphere Kubernetes Engine 2.2.0-1.13.3
- Edge-LB 1.3.0
- Istio 1.0.5

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
