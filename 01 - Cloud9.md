# Cloud9 

## Resize Cloud9 EBS

The default 10GB is quite small when using a docker file for Genomics. Thus, let us resize the EBS volume used by the Cloud9 instance.

To change the EBS volume, please do:

1. Select the Cloud9 instance in the EC2 console deep link to get there
1. Click the root-device link
1. click on the EBS-ID in the box appearing

![ec2 console](https://ec2spotworkshops.com/images/nextflow-on-aws-batch/prerequisites/resize_ebs_0.png)

1. Afterward modify the EBS volume.

![ebs volume](https://ec2spotworkshops.com/images/nextflow-on-aws-batch/prerequisites/resize_ebs_1.png)

And chose a new volume size (e.g. 100GB).

![modify volume](https://ec2spotworkshops.com/images/nextflow-on-aws-batch/prerequisites/resize_ebs_2.png)

## Resize FS

Changing the block device does not increase the size of the file system.

To do so head back to the Cloud9 instance and use the following commands.
```
sudo growpart /dev/xvda 1
sudo resize2fs $(df -h |awk '/^\/dev/{print $1}')
```

The root file-system should now show 99GB.
```
df -h
```
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        483M   60K  483M   1% /dev
tmpfs           493M     0  493M   0% /dev/shm
/dev/xvda1       99G  8.0G   91G   9% /
```

## Update IAM settings for your Workspace

Return to your Cloud9 workspace and click the gear icon (in top right corner) \
Select AWS SETTINGS \
Turn off AWS managed temporary credentials \
Close the Preferences tab \

![cloud9 preference](https://www.eksworkshop.com/images/prerequisites/c9disableiam.png)

Configure our aws cli with our current region as default
```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
```
If jq is not installed, please run the following command
```
sudo yum -y install jq gettext bash-completion moreutils
```

Check if AWS_REGION is set to desired region
```
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
```

Let’s save these into bash_profile
```
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

## Validate the IAM role
Use the GetCallerIdentity CLI command to validate that the Cloud9 IDE is using the correct IAM role.
```
aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
```
If the IAM role is not valid, DO NOT PROCEED. Go to your EventEngine page, click on the AWS Console and copy the Credentials / Cli Snippets. It should look like this
```
export AWS_DEFAULT_REGION=ap-southeast-1 
export AWS_ACCESS_KEY_ID=<ACCESS KEY HERE>
export AWS_SECRET_ACCESS_KEY=<SECRET KEY HERE>
export AWS_SESSION_TOKEN=<AUTH SESSION HERE>
```
Rerun the GetCallerIdentity CLI
