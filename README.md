# kubernetes-ingress

1) Create namespace
---------------------
    apiVersion: v1
    kind: Namespace
    metadata:
        name: node-app

kubectl apply -f ns.yml
-----------------------------
2) create first app deployment

 ---------------------

 apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
  labels:
    env: demo
    type: backend
spec:
  selector:
    matchLabels:
      env: demo
      type: backend
  replicas: 3
  template:
    metadata:
      labels:
        env: demo
        type: backend
    spec:
      containers:
      - name: node-app-container
        image: azhardanish9/my-new-new-express-app:2.0
        ports:
        - containerPort: 3000
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
            

kubectl apply -f deployment.yml
-----------------------------

3) Create first app service

-----------------------------
apiVersion: v1
kind: Service
metadata:
  name: node-app-deployment-svc
spec:
  selector:
    env: demo
    type: backend
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  
kubectl apply -f service.yml
-----------------------------

4) Create second app deployment

---------------------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-docker-app-deployment
  labels:
    app: node-docker
spec:
  selector:
    matchLabels:
      app: node-docker
  replicas: 1
  template:
    metadata:
      labels:
        app: node-docker
    spec:
      containers:
      - name: node-docker-app-container
        image: azhardanish9/node-docker-k8s:2.0
        ports:
        - containerPort: 3015
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m


kubectl apply -f node-docker-deployment.yml
---------------------------

5) Create second app service

----------------------------
apiVersion: v1
kind: Service
metadata:
  name: node-docker-app-svc
spec:
  selector:
    app: node-docker
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 3015
      targetPort: 3015
  

kubectl apply -f node-docker-servic.yml
---------------------------

6) Install aws nginx-ingress-controller

-----------------------------
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/aws/deploy.yaml
------------------------------

7) Create ingress

-----------------------
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-express-app
  namespace: node-app
spec:
  ingressClassName: nginx
  rules:
    - host: my-express-app.local
      http: 
        paths:
          # 1. Routes the root path (/)
          - path: /
            pathType: Prefix
            backend:
              service:
                name: node-app-deployment-svc
                port:
                  number: 3000
          
          # 2. Routes the root path (/orders for 2nd application)
          - path: /orders
            pathType: Prefix
            backend:
              service:
                name: node-docker-app-svc
                port:
                  number: 3015

          
kubectl apply -f ingress.yml

-----------------------
8) sudo nano. /etc/hosts

127.0.0.1   my-express-app.local

and save this file

9) port-forwarding in kind Cluster

    kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80

10) on the browser first app url
    http://my-express-app.local:8080/
    Hello from Kubernetes! Response served by Pod: node-app-deployment-7c66f45555-dgpgf hello aman

    http://my-express-app.local:8080/home
    Welcom to home

11) on the browser second app url
    http://my-express-app.local:8080/orders
    Welcom to orders api

    http://my-express-app.local:8080/orders/about
    This is orders / about page

    