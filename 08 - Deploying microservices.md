# Deploying the microservices

## Deploy NodeJS Backend API
Let’s bring up the NodeJS Backend API!
```
cd ~/environment/ecsdemo-nodejs
```
View the deployment yaml
```
cat kubernetes/deployment.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-nodejs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-nodejs
    spec:
      containers:
      - image: brentley/ecsdemo-nodejs:latest
        imagePullPolicy: Always
        name: ecsdemo-nodejs
        ports:
        - containerPort: 3000
          protocol: TCP
```
Change the image parameter to your very own ECR link; eg:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-nodejs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-nodejs
    spec:
      containers:
      - image: 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-nodejs:latest 
        imagePullPolicy: Always
        name: ecsdemo-nodejs
        ports:
        - containerPort: 3000
          protocol: TCP
```
Apply both the deployment and service yaml files
```
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
```
We can watch the progress by looking at the deployment status:
```
kubectl get deployment ecsdemo-nodejs
```

## Deploy Crystal Backend API
Let’s bring up the Crystal Backend API!
```
cd ~/environment/ecsdemo-crystal
```
View the deployment yaml
```
cat kubernetes/deployment.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-crystal
  labels:
    app: ecsdemo-crystal
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-crystal
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-crystal
    spec:
      containers:
      - image: brentley/ecsdemo-crystal:latest
        imagePullPolicy: Always
        name: ecsdemo-crystal
        ports:
        - containerPort: 3000
          protocol: TCP
```
Change the image parameter to your very own ECR link; eg:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-crystal
  labels:
    app: ecsdemo-crystal
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-crystal
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-crystal
    spec:
      containers:
      - image: 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-crystal:latest 
        imagePullPolicy: Always
        name: ecsdemo-crystal
        ports:
        - containerPort: 3000
          protocol: TCP
```
We can watch the progress by looking at the deployment status:
```
kubectl get deployment ecsdemo-crystal
```

## Check the service type
Before we bring up the frontend service, let’s take a look at the service types we are using: This is kubernetes/service.yaml for our frontend service:
```
apiVersion: v1
kind: Service
metadata:
  name: ecsdemo-frontend
spec:
  selector:
    app: ecsdemo-frontend
  type: LoadBalancer
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 3000
```
Notice **type: LoadBalancer**: This will configure an ELB to handle incoming traffic to this service.

Compare this to kubernetes/service.yaml for one of our backend services:
```
apiVersion: v1
kind: Service
metadata:
  name: ecsdemo-nodejs
spec:
  selector:
    app: ecsdemo-nodejs
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 3000
```
Notice there is no specific service type described. When we check the kubernetes documentation we find that the default type is ClusterIP. This Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster.

## Deploy Frontend Service
Ensure the ELB Service Role exists. We can check for the role and create it if it's missing.
```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```
Let’s bring up the Ruby frontend!
```
cd ~/environment/ecsdemo-frontend
```
View the deployment yaml
```
cat kubernetes/deployment.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-frontend
  labels:
    app: ecsdemo-frontend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-frontend
    spec:
      containers:
      - image: brentley/ecsdemo-frontend:latest
        imagePullPolicy: Always
        name: ecsdemo-frontend
        ports:
        - containerPort: 3000
          protocol: TCP
```
Change the image parameter to your very own ECR link; eg:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-frontend
  labels:
    app: ecsdemo-frontend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-frontend
    spec:
      containers:
      - image: 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:latest 
        imagePullPolicy: Always
        name: ecsdemo-frontend
        ports:
        - containerPort: 3000
          protocol: TCP
```
We can watch the progress by looking at the deployment status:
```
kubectl get deployment ecsdemo-frontend
```

## Find the service address for the Frontend
Now that we have a running service that is type: LoadBalancer we need to find the ELB’s address. We can do this by using the get services operation of kubectl:
```
kubectl get service ecsdemo-frontend
```
```
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP                                                                   PORT(S)        AGE
ecsdemo-frontend   LoadBalancer   10.100.72.179   a88a8874afeac48b58197c741fbd35fd-185901595.ap-southeast-1.elb.amazonaws.com   80:31886/TCP   20h
```
Copy the EXTERNAL-IP of the ecsdemo-frontend and paste it into your browser. You should see a page that looks like the below image \
![demo app](https://www.eksworkshop.com/images/crystal.svg)

## Scale the backend services
When we launched our services, we only launched one container of each. We can confirm this by viewing the running pods:
```
kubectl get deployments
```
Now let’s scale up the backend services:
```
kubectl scale deployment ecsdemo-nodejs --replicas=3
kubectl scale deployment ecsdemo-crystal --replicas=3
```
Confirm by looking at deployments again:
```
kubectl get deployments
```
Also, check the browser tab where we can see our application running. You should now see traffic flowing to multiple backend services.

## Scale the frontend services
```
kubectl get deployments
```
Now let’s scale up the backend services:
```
kubectl scale deployment ecsdemo-frontend --replicas=2
```
Confirm by looking at deployments again:
```
kubectl get deployments
```