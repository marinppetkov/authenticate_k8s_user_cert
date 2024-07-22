# Authenticate to kubernetes using user certificate

1. Generate an user private key and CSR as per this [article](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user), submit the request and approve it. <br>
```
openssl genrsa -out myuser.key 2048
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=podadmin"
```
2. Create a csr.yaml file with the following content:
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: podadmin
spec:
  request: <get the csr content and paste it here>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```
and apply the configuration `kubectl apply -f csr.yaml` <br>

3. Approve the request `kubectl certificate approve podadmin` <br>

4. Create a role and rolebindings with yaml manifests <br>
```
kubectl apply -f developers_role.yaml
kubectl apply -f rolebinding.yaml
```
6. Extract user's signed certificate `kubectl get csr podadmin -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt` <br>

7. Extract the K8S CA cert kubectl `get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"` <br>

8. Configure terraform kuberentes provider configuration with user cert, key and kubernetes CA. <br>
Kuberentes provider [documentation](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#credentials-config)
```
provider "kubernetes" {
  host = "https://cluster_endpoint:port"

  client_certificate     = file("~/.kube/client-cert.pem")
  client_key             = file("~/.kube/client-key.pem")
  cluster_ca_certificate = file("~/.kube/cluster-ca-cert.pem")
}
```

9. Test an example deployment from the terraform folder
