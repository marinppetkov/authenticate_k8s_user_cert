# Authenticate to kubernetes using user certificate

Generate an user private key and CSR as per this [article](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user), submit the request and approve it. <br>
csr.yaml content:
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: podadmin
spec:
  request: <get the csr content and paste it here cat myuser.csr | base64 | tr -d "\n" >
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```
`kubectl apply -f csr.yaml` <br>

Approve the request `kubectl certificate approve podadmin` <br>
Create a role and rolebindings with yaml manifests <br>
Extract user's signed certificate `kubectl get csr podadmin -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt` <br>
Extract the K8S CA cert kubectl `get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"` <br>
Configure terraform kuberentes provider configuration with user cert, key and kubernetes CA. <br>
Kuberentes provider [documentation](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#credentials-config)
```
provider "kubernetes" {
  host = "https://cluster_endpoint:port"

  client_certificate     = file("~/.kube/client-cert.pem")
  client_key             = file("~/.kube/client-key.pem")
  cluster_ca_certificate = file("~/.kube/cluster-ca-cert.pem")
}
```

Test an example deployment
