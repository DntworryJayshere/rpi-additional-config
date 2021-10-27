1. kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.3/manifests/namespace.yaml
2. kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.3/manifests/metallb.yaml
   create configmap for ip reservations
   create secret
3. Install bare-metal nginx-ingress following docs @ https://kubernetes.github.io/ingress-nginx/deploy/ .
   $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.3/deploy/static/provider/baremetal/deploy.yaml
4. Check if the ingress controller pods have started.
   $ kubectl get pods -n ingress-nginx \
   -l app.kubernetes.io/name=ingress-nginx --watch
5. Create nginx loadbalancer.
   $ kubectl apply -f nginx-loadbalancer.yaml

---

$ helm install metallb metallb/metallb --namespace kube-system \
 --set configInline.address-pools[0].name=default \
 --set configInline.address-pools[0].protocol=layer2 \
 --set configInline.address-pools[0].addresses[0]=192.168.1.80-192.168.1.86

$ helm install metallb metallb/metallb -f values

---

$ helm install ingress metallb/metallb --namespace kube-system \
 --set configInline.address-pools[0].name=default \
 --set configInline.address-pools[0].protocol=layer2 \
 --set configInline.address-pools[0].addresses[0]=192.168.1.80-192.168.1.86

---

helm install \
 cert-manager jetstack/cert-manager \
 --namespace cert-manager \
 --create-namespace \
 --version v1.5.4

--set installCRDs=true

---

4. Install cert manager for arm64 architecture.
   curl -sL \
   https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml |\
   sed -r 's/(image:._):(v._)$/\1-arm64:\2/g' > cert-manager-arm.yaml

5. Ensure the image is what we want.
   $ grep image: cert-manager-arm.yaml
6. Create from yaml file created in step 1.
   $ kubectl apply -f cert-manager-arm.yaml
7. Change domain dns on cloudflare to be exposed homenetwork IP.
8. Ensure dns is functioning properly.
   $ dig +short <domain.com>
   //should return exposed ip or cloudflare's proxy Ip adresses
9. Enable port forwarding on home router to direct outside traffic to cluster master ip address on port 80 http, and on port 443 for https.

10. Configure cert-manager to use Lets Encrypt (staging).
    $ kubectl apply -f letsencrypt-issuer-staging.yaml
11. Ensure the new k8s resource type created by cert-manager "ClusterIssuer" status is READY=True.
    $ kubectl get clusterissuers

12. Mannually request test certificate.
    $ kubectl apply -f le-test-certificate.yaml
13. Ensure the certificate status is READY=True.
    $ kubectl get certificates
14. Remove staging certificate & secrets
    $ kubectl delete certificates jmr-devops
    $ kubectl delete secrets jmr-devops-tls

15. Configure cert-manager to use Lets Encrypt (production).
    $ kubectl apply -f letsencrypt-issuer-production.yaml
16. Ensure the new k8s resource type created by cert-manager "ClusterIssuer" status is READY=True.
    $ kubectl get clusterissuers

17. Include proper certificate and ingress config within application yaml.
let
<!-- ---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: jmr-devops
  namespace: default
spec:
  secretName: jmr-devops-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:

- jmr-devops.com

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
annotations:
kubernetes.io/ingress.class: "nginx"
name: personal-site-nginx-ingress
spec:
rules:

- host: jmr-devops.com
  http:
  paths: - path: /
  pathType: Prefix
  backend:
  service:
  name: personal-site-nginx-service
  port:
  number: 80
  tls:
- hosts:
  - jmr-devops.com
    secretName: jmr-devops-tls -->
