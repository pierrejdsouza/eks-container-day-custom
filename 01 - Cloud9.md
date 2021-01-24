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
export AWS_ACCESS_KEY_ID=ASIAS2YPE3ZFU4RD72NQ 
export AWS_SECRET_ACCESS_KEY=uDU3y7BTw9zu/Vr4wMMUrSOEyqSEGZdHxDoq5Z08 
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEBcaCXVzLWVhc3QtMSJHMEUCIQCfCOLD7qoaF5sWZbKdZNAx9jvjCvv2OcB0RkMmlV/u1QIgVHA1gYPSnleuaePTUCOgfbZ0Wi1NQKhVBI7i9vES1zcqoAII8P//////////ARABGgwxOTQ5MTU5MTczODciDKrkrnsad2Nc+INSfyr0AcdH/Q2Z2DSHXv4vEJJhvdx67MFfkOEH1tgDj5AoVBznQTpjU45aJcQGlr7UYK3yiOBwkqMyifiFRJfuxaOejlsHsrygNQxPQ4wFh2u8F0WkRC95P1CIop31Od99weZL8rCxZZP3N28Ri9g8kn2L99NQMvvP3K4I8ApbqxQ1vXw108S3T6PkPBlukpPZB5xM5TTHcK95hts30hnudnWgwHG5FAIX4+9/vLA3tpIZBc69G5p6rpWYLp4DF45msXQrVoW9ckwP7EF/VuHx3eSCwBQ3yHyLiQJAWXht43WwfZUdJYMKkXBWcm1iaWvTqlEWzof+pAIwu5S2gAY6nQGvsueufIvFo3iRG53Ynb3oQYNxeR8DH7R9/9LNpZGTcrSfHbRG/U4UJHC82MdAPhAtCZPYIr53sMkKHD683y2d6FLkFC2Z3yxSlLZLHu/GD05KKryH1KxvrFiXG8562GCo6fVlYrIsl41V1H+uk3IRiy06IqK+8bx9uOCaqwAO/UaOcAM6EntJXRWt0qthRdsI8JwfuE0odeyfr5qO
```
Rerun the GetCallerIdentity CLI
