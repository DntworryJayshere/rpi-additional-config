1. Install bare-metal nginx-ingress following docs @ https://kubernetes.github.io/ingress-nginx/deploy/
   $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.3/deploy/static/provider/baremetal/deploy.yaml
2. Check if the ingress controller pods have started.
   $ kubectl get pods -n ingress-nginx \
   -l app.kubernetes.io/name=ingress-nginx --watch
3. Create nginx loadbalancer.
   $ kubectl apply -f nginx-loadbalancer.yaml

4. Install cert manager for arm64 architecture.
   $ curl -sL https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml | sed -r 's/(image:._):(v._)$/\1-arm64:\2/g' > cert-manager-arm.yaml
5. Ensure the image is what we want.
   $ grep image: cert-manager-arm.yaml
6. Create from yaml file created in step 1.
   $ kubectl apply -f cert-manager-arm.yaml
7. Change domain dns on cloudflare to be exposed homenetwork IP
8. Ensure dns is functioning properly
   $ dig +short <domain.com>
   //should return exposed ip or cloudflare's proxy Ip adresses
9. Enable port forwarding on home router to direct outside traffic to cluster master ip address on port 80 http, and on port 443 for https
