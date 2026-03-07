# Canary-Deployment-Without-ArgoCD-and-HELM

Explanation of Canary Deployment Using NGINX Ingress (Without ArgoCD and Helm)

In this repository, I demonstrate how Canary Deployment works in Kubernetes using the NGINX Ingress Controller without using tools like ArgoCD or Helm.

To implement Canary deployment, we create the following Kubernetes resources:

Two Deployments:
One for the stable version of the application
One for the canary version of the application

Two Services:
One service to expose the stable pods
One service to expose the canary pods

Two Ingress resources:
A primary ingress that routes traffic to the stable service
A canary ingress that routes a percentage of traffic to the canary service using NGINX annotations

The canary traffic distribution is controlled using the annotation:
nginx.ingress.kubernetes.io/canary: "true"
nginx.ingress.kubernetes.io/canary-weight: "15"

This means:
85% traffic → stable service
15% traffic → canary service


Prerequisites

Before applying the manifests, the following components must be available in the cluster:
1. A running Kubernetes cluster
2. NGINX Ingress Controller installed (kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/aws/deploy.yaml -> To install ingress controller)
3. A LoadBalancer or NodePort service created by the ingress controller.

Verification;

After deploying all the resources, we can verify the traffic split by checking the Ingress controller logs.

Run a command to send traffic to our pods: 

for i in {1..20}; do curl <your end-pointurl>;done

ex:
for i in {1..50}; do curl " http://a65cbd064f24948c98cc7eb129f9fef2-6b14e52a4b3b5e58.elb.ap-south-1.amazonaws.com";done

kubectl logs -n ingress-nginx <ingress-controller-pod>

in the logs we can see like this: 
[default-myapp-stable-service-80]
[default-myapp-canary-service-80]
192.168.x.x


This indicates:
which service handled the request
which pod IP served the request
So we can confirm that traffic is being split between stable and canary services.


IMP POINTS:

There is no new replica created during canary deployment, Ingress only splits traffic, it does not create new pods.


Stable pods = 3
Canary pods = 1
canary-weight = 50

So here stabe pods is 3 so these 3 pods receive 50% traffic and the canary pod is 1 and this 1 pod receives 50% traffic.

So in production rollout usually follows this pattern:
Step 1 → Deploy canary (1 pod)
Step 2 → Send 5% traffic
Step 3 → Increase canary replicas
Step 4 → Increase traffic gradually
Step 5 → Promote canary to stable.




