# ECR and Docker images

## Create the repository
1. Go to the ECR page from the AWS Management Console.

2. On the left hand navigation, click on **Repositories**

3. Click on **Create repository**

4. Visibility should be set to private

5. The repository name should be ecsdemo-frontend

6. Click **Create repository**

7. Repeat the steps for ecsdemo-nodejs and ecsdemo-crystal

## Building the microservices docker images

1. Let's start with the ecsdemo-frontend:
```
cd ecsdemo-frontend/
```

2. Build the docker image:
```
docker build -t ecsdemo-frontend .
```

3. Tag the docker image (Please use your own ECR link by going to the ECR page)
```
docker tag ecsdemo-frontend:latest 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:latest
```

4. Push the docker image to ECR
```
docker push 194915917387.dkr.ecr.ap-southeast-1.amazonaws.com/ecsdemo-frontend:latest
```

5. Repeat these steps for ecsdemo-nodejs and ecsdemo-crystal
6. Check and verify that your image has been uploaded/pushed to the ECR repository. You can do this by going to the ECR page in the AWS Management Console.

If you get an error message saying you have "No auth credentials", do the following
```
aws ecr get-login
```
Copy the output of the previous command; example:
```
docker login -u AWS -p eyJwYXlsb2FkIjoia0pYZXY1dUo4aU0rMTBnZ01wdnhXVjFYRjROaDlQN0YvNnRzVVV6bHVrcWpraXhYRExVVkdLRDRMNnhqSTVQQjdTWGRwcmJRcUNnMmVWNEtoQ082SytSTWh6M1E3c3JicExHcUVBY29UMkx2UlpFWEREYmtpVHowZyt6VEYwK0Rpa1pFTG1ka1dPcGZVUk5Yb0ZkV0d6TDIvd0ZYQlZEQlZud0NobWVBRzFxaGs1VGcwM3FMTW5wck1EZk5jUlVaSllUM2IyZ3BCZUk3MDZZdlRNSGVEMU1TN1FRWUgyeVZNbjhHaTQxbDBpazcrbnNPdkdIUWNnT2hFYnlGUVAwc1kvWnVNRVBqUmhoMlUzTUJrSnpOWGIrc0FIM0gvUVRUdERNck1QbnBZd3RwZXVYMDc2UXppNnRpcVhjdlZ0VTZiVkVKWGVUU0pqVzdGTE5jOVVLaUtoeXl3N1pmMUVpRXlvL0RwS0I5M0FzVWtOanFmZHg5YkFUeVd2QlB3Wjd5bTN0aVJpMnBtTVFVb1NnNmVhMitUMFpaWjV5My9RK09nOEtrbENLR3l5S2hxVmg1REtMbWJVMUpLVFBaaW1Xa1M5Vk02QkFhUWx3NXhSVDFKczN4WFEvZktPWkxIbktLN2FQaVNiR1UrY1dkKzhsWm16NWhIeGkrMjRJSis1dEFqM2NlNXBnL2lqemdrSFlnL25OeXUyZGIwNlQxei9HNExuTG1iSGsrdDVTV0k2Uy9vN3piLzk5ek9EdkQvVWlTbS9yK1c1cTVYMjZhOERmM2dYYUFpdENkemRMeDJVczNCbiswMHlUVzVndk1GQWx4UE5Zb0NOcUZKSGJuQmhwWFFuOHMwWDJma2FwWC8rbHQwNHJUNDEwRmxGeWxveFJ2MmRYZnBOZ3ZrclpvaDNpWnZIaVZNU3lJMlloUFFxSHJIWU9jSXFzTkQxM3pBTEdkVllCTFUxbU1ScytuVHRYU2NOZm9ZdllPbW1qdXZabE93T0JjMGhzalBCZVNub1VLaXlSOHF6azdKU2dOVitnOFlUTW5Dc1FobUNSNXU5UWEzVWdFbTNZS3BoY0NyWlVUVVpLNVYzRk9aZGFkeFR2SHdtcVNTZ094ZEkrOUNMOEVaYStoa1lFQmQzbGVNK01USDJrb3ZIMHJTaEZhK1dkdDlVbUk2QXg1Y3JNbWxqdDJxVGU4Vm5NNFA1WHg1Z0dSamJmdVRUL1VJYVVIWmJDUVY0SG1yY1JVRloyb1Jubkh2MmxKMlhxOHZ1L1l1VVUzalZpYzVBVzRJMVMwVUR6YUwweXNZSmFpUDJYaHQ0Nzg3anZOMnhBaUdlVDdzdFlmeS9acEpKdmNnRFB2QjYxNkNVelFoYysvYldjckxSWWpJVWhhQ3BzY3N0djJ1aUdhNjJYalVQYnUvY0p3eWlOL011azRIbGppRi9yYWZpZU1KWkUzMHJYZ1hSUnlrOUZ2Rll4WkVPWDZ2Q3R2NnhhaFhlbnNsUFdMRVN3VDBkZi85NXB6S3loZTBET3RaZm9TOU5kZ3pBT0RXdi9IVUgxeTMzcXlKYy9XZk1pZkEwWXVxM1dEZmRWWk5tdTZGY3N5VUt6eGlpUXl5QXdWZEU5Mk5rVHprOWd4VlNKTEZqU1RzUGZqTldqZjg5M01CVzNNakl1VWRpZHlYbFdwWDcxTVJmaUZETjJjYTlVeHM1MjhIeVlQaFAwQW9vcUFVOXQxdFYwUmx5TlppQ3Rna2xyajdLSFFDekh2Vjl2eDZSU0FPRWtObWRxZ0lsS3pyblQyZ3JWREFwREN6d09IUlFzcWlTbkd6TWlyVVV6OWs1NkFoQ1pOaDRoZG9ZVUhvWHdTZG1tUzhNSTNHUlF2M0RRcmNEZm1RelE0IiwiZGF0YWtleSI6IkFRRUJBSGlkRXJaQ2ZoS09lRE0wOCtjUDVmdHlqdlE5WE1NU1E0cEswRlpudkFaWEpnQUFBSDR3ZkFZSktvWklodmNOQVFjR29HOHdiUUlCQURCb0Jna3Foa2lHOXcwQkJ3RXdIZ1lKWUlaSUFXVURCQUV1TUJFRURQOXF0NzI0a2JSbEUyVFZ4Z0lCRUlBN1AxYThDZExzRVppc1E2cUJyaVV5Q2xGaGc5U0ZGdmVkUlFZVkNwT0dqMDNiRXc2TXlHd2VlMXF4cmxvMnVRb1IvK1hONGdRNCtXQlhIMFE9IiwidmVyc2lvbiI6IjIiLCJ0eXBlIjoiREFUQV9LRVkiLCJleHBpcmF0aW9uIjoxNjExNDc4MzUxfQ== https://194915917387.dkr.ecr.ap-southeast-1.amazonaws.com
```
