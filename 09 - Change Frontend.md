# Running a new version for Frontend
The auto-reload seems to be fast, and I dont like the color grey. Let's make some changes and deploy a new frontend

## Making changes to the Frontend
1. Go to the frontend directory
```
cd ~/environment/ecsdemo-frontend
```
2. Change the color of the background
```
vim app/views/application/index.html.erb 
```
```
background-color: #666666;
```
```
background-color: #FFFFFF;
```
3. Change the refresh interval from 50 to 100
```
vim app/views/layouts/application.html.erb 
```
```
setTimeout(function() {location=''}, 50)
```
```
setTimeout(function() {location=''}, 100)
```

## Build a new image for Frontend
1. Now that we've already made some changes to our frontend, let's build a new version of the image.
```
cd ~/environment/ecsdemo-frontend
```
2. Build the docker image:
```
docker build -t ecsdemo-frontend .
```

3. Tag the docker image (Please use your own ECR link by going to the ECR page) Notice that I've changed the version at the end to v2.0.0
```
docker tag ecsdemo-frontend:latest 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:v2.0.0
```

4. Push the docker image to ECR
```
docker push 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:v2.0.0
```

## Deploying the new Frontend
1. Set the image of the frontend deployment
```
kubectl set image deployment ecsdemo-frontend ecsdemo-frontend=194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:v2.0.0
```

2. Do a rolling restart for the frontend deployment
```
kubectl rollout restart deployment ecsdemo-frontend
```

3. Check your browser for the changes to take effect.

