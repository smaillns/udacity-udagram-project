# Udacity Cloud Devleloper
## Project : Refactor Udagram App into Microservices and Deploy
Standing up a kubernetes cluster on AWS using kops and Travis-CI as CI/CD Pipeline 

#### Step0: Prepare the application to deploy 
You can find the source code and the different Dockerfile in the directory **udagram-prject**

#### Step1: Deploy the cluster

Update and load the required environment variables:

The domain name that is hosted in AWS Route 53
```bash
export DOMAIN_NAME="kops.ucci.uk"
```
Friendly name to use as an alias for our cluster
```bash
export CLUSTER_ALIAS="udagramk8s"
```
Full DNS name of our cluster
```bash 
export CLUSTER_FULL_NAME="${CLUSTER_ALIAS}.${DOMAIN_NAME}"
```

AWS availability zone where the cluster will be created
```bash
export CLUSTER_AWS_AZ="us-east-1a"
```
AWS Route 53 hosted zone ID for your domain
```bash
export DOMAIN_NAME_ZONE_ID=$(aws route53 list-hosted-zones \
       | jq -r '.HostedZones[] | select(.Name=="'${DOMAIN_NAME}'.") | .Id' \
       | sed 's/\/hostedzone\///') 
```

**Note :** you have to configure the aws profile first ```aws configure```, the IAM must the following policies attached to it
```
    AmazonEC2FullAccess
    AmazonRoute53FullAccess
    AmazonS3FullAccess
    IAMFullAccess
    AmazonVPCFullAccess
```



We use **Kops** to stand up the cluster

Create the S3 bucket in AWS, which will be used by Kops for cluster configuration storage:
```bash
aws s3api create-bucket --bucket ${CLUSTER_FULL_NAME}-state --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
```

Set the KOPS_STATE_STORE variable to the URL of the S3 bucket that was just created:
```bash
export KOPS_STATE_STORE="s3://${CLUSTER_FULL_NAME}-state"
```
Create the cluster with Kops:
```bash
kops create cluster \
     --name=${CLUSTER_FULL_NAME} \
     --zones=${CLUSTER_AWS_AZ} \
     --master-size="t2.medium" \
     --node-size="t2.medium" \
     --node-count="2" \
     --dns-zone=${DOMAIN_NAME} \
     --ssh-public-key="~/.ssh/id_rsa.pub" \
```
```bash
kops update cluster --name udagramk8s.kops.ucci.uk --yes
```
It will take approximately 5 minutes for the cluster to become availble, next we validate and check the status of
 the cluster by running the following command :
```bash
kops validate cluster
```
Check more if the cluster is ready :
```bash
kubectl get nodes
```




#### Step2 : Configure the Travis-CI pipeline


[pip](https://pip.pypa.io/en/stable/) 