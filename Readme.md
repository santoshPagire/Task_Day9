## Project Overview
## Stage 1: Setting Up the Kubernetes Cluster and Static Web App
Set Up Minikube:
Ensure Minikube is installed and running on the local Ubuntu machine.
Verify the Kubernetes cluster is functioning correctly.
![alt text](<image/Screenshot from 2024-07-18 16-40-34.png>)
## Deploy Static Web App:
+ Create a Dockerfile for a simple static web application (e.g., an HTML page served by Nginx).
+ Build a Docker image for the static web application.

Create index.html
```bash
<!doctype html>
<html>
 <body>
    <head>
     <title>Docker Project</title>
    </head>
    <body>
     <p>Welcome to my Docker Project!<p>
        </body>
</html>
```
Create Dockerfile
```bash
FROM nginx:1.10.1-alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
Build image:
```bash
docker build santoshpagire/customnginx-app .
```
Push the Docker image to Docker Hub or a local registry.
```bash
docker push santoshpagire/customnginx-app:latest
```
![alt text](<image/Screenshot from 2024-07-18 17-30-18.png>)
![alt text](<image/Screenshot from 2024-07-18 17-35-18.png>)
## Kubernetes Deployment:
+ Write a Kubernetes deployment manifest to deploy the static web application.
+ Write a Kubernetes service manifest to expose the static web application within the cluster.
+ Apply the deployment and service manifests to the Kubernetes cluster.
> Create frontend-delpoyment.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: nginx
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: santoshpagire/customnginx-app:latest
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: 50m
            requests:
              cpu: 20m

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  
  ports:
    - protocol: TCP
      port: 80  
      targetPort: 80  
  type: NodePort
```

Create backtend-delpoyment.yml
```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
          - "-text= This is test message from backend"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678

```
Apply the deployment and service manifests to the Kubernetes cluster.

```bash
kubectl apply -f frontend-delpoyment.yml

kubectl apply -f backend-delpoyment.yml
```

## Stage 2: Configuring Ingress Networking
Install and Configure Ingress Controller:
```bash
minikube start --addons=ingress
```
Install an ingress controller (e.g., Nginx Ingress Controller) in the Minikube cluster.
Verify the ingress controller is running and accessible.

```bash
minikube addons enable ingress

kubectl get pods -n kube-system
```
Create Ingress Resource:
+ Write an ingress resource manifest to route external traffic to the static web application.
+ Configure advanced ingress rules for path-based routing and host-based routing (use at least two different hostnames and paths).
+ Implement TLS termination for secure connections.
+ Configure URL rewriting in the ingress resource to modify incoming URLs before they reach the backend services.
+ Enable sticky sessions to ensure that requests from the same client are directed to the same backend pod.

> create ingress-resource.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: static-web-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
spec:
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /home
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    - host: myapp.com
      http:
        paths:
          - path: /page
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
  tls:
    - hosts:
        - myapp.com
      secretName: tls-secret
```
Apply the  ingress-resource.yaml
```bash
kubectl apply -f ingress-resource.yaml
```
Check output using 
```bash
https://myapp.com/home
https://myapp.com/page
```
Results:
![alt text](<image/Screenshot from 2024-07-19 14-22-50.png>)
![alt text](<image/Screenshot from 2024-07-19 14-23-19.png>)

## Stage 3: Implementing Horizontal Pod Autoscaling
+ Configure Horizontal Pod Autoscaler:
+ Write a horizontal pod autoscaler (HPA) manifest to automatically scale the static web application pods based on CPU utilization.
+ Set thresholds for minimum and maximum pod replicas.
### Stress Testing:
+ Perform stress testing to simulate traffic and validate the HPA configuration.
+ Monitor the scaling behavior and ensure the application scales up and down based on the load.
Deliverables:
+ Horizontal pod autoscaler YAML file
> Create hpa.yaml
```bash
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 5

```
 Documentation or screenshots of the stress testing process and scaling behavior
![alt text](<image/Screenshot from 2024-07-19 12-23-14.png>)
![alt text](<image/Screenshot from 2024-07-19 14-40-34.png>)

## Stage 4: Final Validation and Cleanup
### Final Validation:
Validate the ingress networking, URL rewriting, and sticky sessions configurations by accessing the web application through different hostnames and paths.
![alt text](<image/Screenshot from 2024-07-19 14-22-50.png>)
![alt text](<image/Screenshot from 2024-07-19 14-23-19.png>)
Verify the application's availability and performance during different load conditions.
![alt text](<image/Screenshot from 2024-07-19 14-40-05.png>)
![alt text](<image/Screenshot from 2024-07-19 14-40-34.png>)
### Cleanup:
Provide commands or scripts to clean up the Kubernetes resources created during the project (deployments, services, ingress, HPA).
Delete deployments:
```bash
kubectl delete deployment frontend
kubectl delete deployment backend
```
Delete Services:
```bash
kubectl delete service frontend-service
kubectl delete service backend-service

```
Delete ingress:
```bash
kubectl delete ingress static-web-app-ingress
```
Delete hpa:
```bash
kubectl delete hpa nginx-app-hpa
```

