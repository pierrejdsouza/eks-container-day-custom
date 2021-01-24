# Autoscaling

## Scale the frontend application
1. Change the resources for the frontend deployment
```
kubectl set resources deploy ecsdemo-frontend --requests=cpu=100m
```
2. Do a rolling restart for this to take effect.
```
kubectl rollout restart deployment ecsdemo-frontend
```
3. Create a new horizal pod autoscaler (HPA) for the frontend deployment
```
kubectl autoscale deployment ecsdemo-frontend --cpu-percent=10 --min=1 --max=10
```
4. View the HPA using kubectl. You probably will see <unknown>/10% for 1-2 minutes and then you should be able to see 0%/10%
```
kubectl get hpa
```
5. Generate load to trigger scaling. Open a new terminal tab in your Cloud9 environment, and execute a while loop
```
while true; do curl -Lvso /dev/null http://a88a8874afeac48b58197c741fbd35fd-185901595.ap-southeast-1.elb.amazonaws.com/; done
```
6. In the other tab, watch the HPA with the following command
```
kubectl get hpa -w
```
7. You will see HPA scale the pods from 1 up to our configured maximum (10) until the CPU average is below our target (10%)
8. You can now stop (Ctrl + C) load test that was running in the other terminal. You will notice that HPA will slowly bring the replica count to min number based on its configuration. You should also get out of load testing application by pressing Ctrl + D.

## Configure Cluster Autoscaler (CA)
Cluster Autoscaler for AWS provides integration with Auto Scaling groups. It enables users to choose from four different options of deployment:

One Auto Scaling group
Multiple Auto Scaling groups
Auto-Discovery
Control-plane Node setup
Auto-Discovery is the preferred method to configure Cluster Autoscaler. Click here for more information.

Cluster Autoscaler will attempt to determine the CPU, memory, and GPU resources provided by an Auto Scaling Group based on the instance type specified in its Launch Configuration or Launch Template.

### Configure the ASG
You configure the size of your Auto Scaling group by setting the minimum, maximum, and desired capacity. When we created the cluster we set these settings to 3.
```
aws autoscaling \
    describe-auto-scaling-groups \
    --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].[AutoScalingGroupName, MinSize, MaxSize,DesiredCapacity]" \
    --output table
```
```
-------------------------------------------------------------
|                 DescribeAutoScalingGroups                 |
+-------------------------------------------+----+----+-----+
|  eks-1eb9b447-f3c1-0456-af77-af0bbd65bc9f |  3 |  3 |  3  |
+-------------------------------------------+----+----+-----+
```
Now, increase the maximum capacity to 4 instances
```
# we need the ASG name
export ASG_NAME=$(aws autoscaling describe-auto-scaling-groups --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].AutoScalingGroupName" --output text)

# increase max capacity up to 4
aws autoscaling \
    update-auto-scaling-group \
    --auto-scaling-group-name ${ASG_NAME} \
    --min-size 3 \
    --desired-capacity 3 \
    --max-size 4

# Check new values
aws autoscaling \
    describe-auto-scaling-groups \
    --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].[AutoScalingGroupName, MinSize, MaxSize,DesiredCapacity]" \
    --output table
```
### IAM roles for service accounts
With IAM roles for service accounts on Amazon EKS clusters, you can associate an IAM role with a Kubernetes service account. This service account can then provide AWS permissions to the containers in any pod that uses that service account. With this feature, you no longer need to provide extended permissions to the node IAM role so that pods on that node can call AWS APIs.

Enabling IAM roles for service accounts on your cluster
```
eksctl utils associate-iam-oidc-provider \
    --cluster eksworkshop-eksctl \
    --approve
```
Creating an IAM policy for your service account that will allow your CA pod to interact with the autoscaling groups.
```
mkdir ~/environment/cluster-autoscaler

cat <<EoF > ~/environment/cluster-autoscaler/k8s-asg-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EoF

aws iam create-policy   \
  --policy-name k8s-asg-policy \
  --policy-document file://~/environment/cluster-autoscaler/k8s-asg-policy.json
```
Finally, create an IAM role for the cluster-autoscaler Service Account in the kube-system namespace.
```
eksctl create iamserviceaccount \
    --name cluster-autoscaler \
    --namespace kube-system \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn "arn:aws:iam::${ACCOUNT_ID}:policy/k8s-asg-policy" \
    --approve \
    --override-existing-serviceaccounts
```
Make sure your service account with the ARN of the IAM role is annotated
```
kubectl -n kube-system describe sa cluster-autoscaler
```
Output
```
Name:                cluster-autoscaler
Namespace:           kube-system
Labels:              <none>
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::197520326489:role/eksctl-eksworkshop-eksctl-addon-iamserviceac-Role1-12LNPCGBD6IPZ
Image pull secrets:  <none>
Mountable secrets:   cluster-autoscaler-token-vfk8n
Tokens:              cluster-autoscaler-token-vfk8n
Events:              <none>
```
## Deploy the Cluster Autoscaler (CA)
1. Deploy the Cluster Autoscaler to your cluster with the following command.
```
kubectl apply -f https://www.eksworkshop.com/beginner/080_scaling/deploy_ca.files/cluster-autoscaler-autodiscover.yaml
```
2. To prevent CA from removing nodes where its own pod is running, we will add the cluster-autoscaler.kubernetes.io/safe-to-evict annotation to its deployment with the following command
```
kubectl -n kube-system \
    annotate deployment.apps/cluster-autoscaler \
    cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```
3. Finally let’s update the autoscaler image
```
# we need to retrieve the latest docker image available for our EKS version
export K8S_VERSION=$(kubectl version --short | grep 'Server Version:' | sed 's/[^0-9.]*\([0-9.]*\).*/\1/' | cut -d. -f1,2)
export AUTOSCALER_VERSION=$(curl -s "https://api.github.com/repos/kubernetes/autoscaler/releases" | grep '"tag_name":' | sed -s 's/.*-\([0-9][0-9\.]*\).*/\1/' | grep -m1 ${K8S_VERSION})

kubectl -n kube-system \
    set image deployment.apps/cluster-autoscaler \
    cluster-autoscaler=us.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v${AUTOSCALER_VERSION}
```
4. Watch the logs
```
kubectl -n kube-system logs -f deployment/cluster-autoscaler
```
5. We are now ready to scale our cluster!

## Scale cluster with CA
1. Change the resources for the nodejs deployment
```
kubectl set resources deploy ecsdemo-nodejs --requests=cpu=500m
```
2. Do a rolling restart for this to take effect.
```
kubectl rollout restart deployment ecsdemo-nodejs
```
3. Scale the ecsdemo-nodejs replicaset
```
kubectl scale deployment ecsdemo-nodejs --replicas=10
```
4. Some pods will be in the Pending state, which triggers the cluster-autoscaler to scale out the EC2 fleet.
```
kubectl get pods -l app=nginx -o wide --watch
```
```
NAME                                 READY     STATUS    RESTARTS   AGE

ecsdemo-nodejs-7cb554c7d5-2d4gp   0/1       Pending   0          11s
ecsdemo-nodejs-7cb554c7d5-2nh69   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-45mqz   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-4qvzl   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-5jddd   1/1       Running   0          34s
ecsdemo-nodejs-7cb554c7d5-5sx4h   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-5xbjp   0/1       Pending   0          11s
ecsdemo-nodejs-7cb554c7d5-6l84p   0/1       Pending   0          11s
ecsdemo-nodejs-7cb554c7d5-7vp7l   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-86pr6   0/1       Pending   0          12s
ecsdemo-nodejs-7cb554c7d5-88ttw   0/1       Pending   0          12s
```
5. View the cluster-autoscaler logs
```
kubectl -n kube-system logs -f deployment/cluster-autoscaler
```
6. You will notice Cluster Autoscaler events similar to below \
![cluster-autoscaler](https://www.eksworkshop.com/images/scaling-asg-up2.png)
7. Check the EC2 AWS Management Console to confirm that the Auto Scaling groups are scaling up to meet demand. This may take a few minutes. You can also follow along with the pod deployment from the command line. You should see the pods transition from pending to running as nodes are scaled up. \
![node-ec2](https://www.eksworkshop.com/images/scaling-asg-up.png)
8. You may use kubectl as well
```
kubectl get nodes
```
```
ip-192-168-12-114.ap-southeast-1.compute.internal   Ready    <none>   3d6h   v1.17.7-eks-bffbac
ip-192-168-29-155.ap-southeast-1.compute.internal   Ready    <none>   63s    v1.17.7-eks-bffbac
ip-192-168-55-187.ap-southeast-1.compute.internal   Ready    <none>   3d6h   v1.17.7-eks-bffbac
ip-192-168-82-113.ap-southeast-1.compute.internal   Ready    <none>   8h     v1.17.7-eks-bffbac
```