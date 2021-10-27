An example Ingress that makes use of the controller:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
annotations:
kubernetes.io/ingress.class: nginx
name: example
namespace: foo
spec:
ingressClassName: example-class
rules: - host: www.example.com
http:
paths: - path: /
pathType: Prefix
backend:
service:
name: exampleService
port: 80 # This section is only required if TLS is to be enabled for the Ingress
tls: - hosts: - www.example.com
secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

apiVersion: v1
kind: Secret
metadata:
name: example-tls
namespace: foo
data:
tls.crt: <base64 encoded cert>
tls.key: <base64 encoded key>
type: kubernetes.io/tls

helm install \
 cert-manager jetstack/cert-manager \
 --namespace cert-manager \
 --create-namespace \
 --version v1.5.4

--set installCRDs=true
