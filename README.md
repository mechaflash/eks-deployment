# EKS-deployment

## Pre-Requisites
You will need to have an AWS IAM user with admin privileges and CLI access, as well as aws-cli, kubectl, and eksctl installed on either your local machine or an EC2 instance to run the appropriate commands to perform an EKS cluster deployment using the methods to follow. eksctl makes it very easy to create and manage eks clusters, creating the cloudformation stack and all necessary resources for your cluster within a single command.

### AWS IAM User

#### Via aws-cli
`aws iam create-user --user-name <user-name>`   
`aws iam create-access-key --user-name <user-name>`   
*NOTE: Save AccessKeyId and SecretAccessKey from the output from `aws iam create-access-key`*   
`aws iam attach-user-policy --user-name <user-name> --policy-arn arn:aws:iam::aws:policy/AdministratorAccess`   
NOTE: The above provides full admin access to the user. If you wish to create a user with more restricted access rights, you may create a custom IAM policy and attach it via the ARN, replacing the --policy-arn value above.


### Using an amazon linux EC2 instance
#### Create an EC2 Instance
##### Via aws-cli
###### Create key-pair
`aws ec2 create-key-pair --key-name my-k8-server-key --query 'KeyMaterial' --output text > my-k8-server-key.pem`   

Update the key's permissions.

Linux:
`chmod 400 my-k8-server-key`

Windows:
Right-click on the key file and go to properties.
Navigate to Security > Advanced
'Disable inheritance' if it is enabled.
Remove all permission entries, except for 'Full control' access for your user.

###### Create Security Group
Retrieve the VPC ID that you will need this security group deployed for.
To retrieve the VPC by VPC name and region:
`aws ec2 describe-vpcs --filters Name='tag:Name',Values='<VPC NAME>' --region <REGION> --output=text | find /i "vpc-"`
NOTE: Replace <VPC NAME> and <REGION> appropriately.
      Record the vpc-id from the output.
`aws ec2 create-security-group --group-name ssh-sg --description "Allow SSH" --vpc-id <VPC ID>`
NOTE: Replace the <VPC ID> from the previous command.
      Record the `GroupId` from the output.

`aws ec2 authorize-security-group-ingress --group-id <your_security_group_id> --protocol tcp --port 22 --cidr 0.0.0.0/0`
NOTE: You can limit the `--cidr` field to your own public IPV4 address to limit access.

###### Get Latest Amazon Linux 2 AMI
`aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-2.0.????????.?-x86_64-gp2" "Name=state,Values=available" "Name=virtualization-type,Values=hvm" --region <REGION> --query "reverse(sort_by(Images, &CreationDate))[0].ImageId"`
NOTE: Replace <REGION> appropriately.
Record the output to use in the EC2 instance creation for the AMI image to use.

###### Launch the EC2 Instance
TO-DO: Below command will not work without specifying a subnet from a VPC you wish to launch the instance into.
`aws ec2 run-instances --image-id <AMI_ID> --count 1 --instance-type t2.micro --key-name my-k8-server-key --security-group-ids <your_security_group_id>'`
NOTE: Replace <AMI_ID> from the output of the previous step.
Replace Replace <your_security_group_id> from the output of the Create Security Group step.

#### Upgrading aws-cli
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html   
Instructions on installing can be found in the above link (subject to change, so will not document exact steps here).   
NOTE: There are times when running `aws --version` shows the older version. After you perform the update, close and reconnect to the EC2 instance and re-run `aws --version`.

#### Installing kubectl
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html   
Instructions on installing can be found in the above link (subject to change, so will not document exact steps here).   
NOTE: Ensure to select the appropriate kubectl version with kubernetes cluster version you will be managing.

#### Installing eksctl
https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html   
Instructions on installing can be found in the above link (subject to change, so will not document exact steps here).

## Using eksctl to Deploy a Cluster
From either your computer or the EC2 you've setup to deploy the cluster from, you can use eksctl to perform a cluster deployment. Below is an example command:   
`eksctl create cluster --name dev --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes-min 1 --nodes-max 4 --managed`   
NOTE: Replace the fields accordingly with your own values.   
This will create a cloud-formation stack with a name with format `eksctl-<name>-cluster`   
This cloud formation stack will take approximately 15 minutes to deploy all necessary resources for the eks cluster.   
NOTE: Sometimes the deployment will fail due to a lack of resources for the region you're deploying the cluster in. If this occurs, you will need to specify AZs that were not included in the error message to the eksctl command above.
