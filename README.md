# Setup RBAC with MKE based k8s installtion

## Prerequisites

### This installation guide was tested with the following components:

- Mesosphere DC/OS Enterprise 1.12.3 or higher
- Mesosphere Kubernetes Engine 2.2.0-1.13.3 or higher
- Edge-LB 1.3.0 or higher 
- Access to K8s cluster running in MKE with Aurhorization mode as `RBAC` find the link below on how to do that.
  https://docs.mesosphere.com/services/kubernetes/2.3.3-1.14.3/operations/authn-and-authz/#rbac

1. Validate that you have admin access to k8s api-server:

```bash
# Validate that you can run kubectl against the api-server.
kubectl get pods -n kube-system

# Check that you have admin access to the k8s cluster.
kubectl config view
```

2. Provision a service account for a an individual member ex john-smith
```bash
kubectl create serviceaccount john-sa
```

3. Bind the service account to the appropriate roles to grant privileges for actions you desire. In this case its view permissions
```bash
#Below command bind a default cluster-role with service account we created above and associate it with default namespace.
kubectl create clusterrolebinding john-sa-binding --clusterrole=edit --serviceaccount=default:john-sa
```
#### Note
`1. Whenever we create a service account, K8s api server creates a token for this service account by default so that this Service account can authenticate itself to the api-server. We are going to extract this token to access the cluster for external access.
2. Above example uses default SA use this link to get creative and set up further custome roles as per your needs 
https://kubernetes.io/docs/reference/access-authn-authz/rbac/
`

4. Retrieve the token
      ```bash
      a. # Verify the secrets exist
      kubectl get secrets
      ```
      that should give an output something like this.
      ```bash
      NAME                                  TYPE                                  DATA      AGE
      default-token-2s56x                   kubernetes.io/service-account-token   3         30d
      k8s-nginx-ingress-token-h49rc         kubernetes.io/service-account-token   3         21h
      john-sa-token-rq4ls                   kubernetes.io/service-account-token   3         12m
      ```

      ```bash
      b. # Verify the token generated for service account by describing the secret 
      kubect describe secret john-sa-token-rq4ls
      ```
      ouput should be something similar to below 
      ```bash
      Name:         john-sa-token-rq4ls
      Namespace:    default
      Labels:       <none>
      Annotations:  kubernetes.io/service-account.name=john-sa
                    kubernetes.io/service-account.uid=aa1c318a-bc3d-11e8-b171-023b9d05d78
      
      Type:  kubernetes.io/service-account-token
      
      Data
      ====
      ca.crt:     33605 bytes
      namespace:  7 bytes
      token:      eyJhb3ciOi . . . [output snipped]
      ```
      ```bash
      c. #You can put the token into an environment variable, which provides a convenient way to access it when OS = MAC, make sure to add your secret name instead the one-mentioned below
      export TOKEN=$(kubectl get secret john-sa-token-rq4ls -o=jsonpath="{.data.token}" | base64 -D -i -)

      # when OS = Linux
      export TOKEN=$(kubectl get secret john-sa-token-rq4ls -o=jsonpath="{.data.token}" | base64 -d -i -)
      ```


5. Configure your kubeconfig file to include the service account you included. Below is the sample and make to include the token you generate to the token section.

```bash
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://X.X.X.X:6443
  name: kubernetes-cluster1
contexts:
- context:
    cluster: kubernetes-cluster1
    user: kubernetes-cluster1
  name: kubernetes-cluster1
- context:
    cluster: kubernetes-cluster1
    user: john-sa
  name: john-sa
current-context: kubernetes-cluster1
kind: Config
preferences: {}
users:
- name: kubernetes-cluster1
  user:
    token:  XXXXXXXXXXXXXXX
- name: john-sa
  user:
    token: XXXXXXXXXXXXXXXXXXXX

```

6. Once you have updated the kubeconfig file. You can validate the access like below
```bash
# Verify the context is updated below command should switch the context to your service account 
kubectx john-sa

# Run commands to test 
kubectl auth can-i get deployments

kubectl get pods 

```
7. You can also use the token to make http calls to k8s api
```bash
curl -H "Authorization: Bearer $TOKEN" https://api.cluster-address/api/v1/pods -k
```