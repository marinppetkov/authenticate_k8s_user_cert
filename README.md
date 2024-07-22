# Authenticate to kubernetes using user certificate

Generate an user private key and CSR as per this [article](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user), submit the request and approve it. <br>
Extract the K8S CA cert kubectl `get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"` <br>
Create a role and rolebindings with yaml manifests <br>
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
