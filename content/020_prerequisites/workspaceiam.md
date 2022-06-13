---
title: "Pre-requisites"
chapter: false
weight: 19
---

The following command adds more disk space to the root volume of the EC2 instance that Cloud9 runs on. Once the command completes, we reboot the instance and it could take a minute or two for the IDE to come back online.

```sh
pip3 install --user --upgrade boto3
export instance_id=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
python -c "import boto3
import os
from botocore.exceptions import ClientError 
ec2 = boto3.client('ec2')
volume_info = ec2.describe_volumes(
    Filters=[
        {
            'Name': 'attachment.instance-id',
            'Values': [
                os.getenv('instance_id')
            ]
        }
    ]
)
volume_id = volume_info['Volumes'][0]['VolumeId']
try:
    resize = ec2.modify_volume(    
            VolumeId=volume_id,    
            Size=30
    )
    print(resize)
except ClientError as e:
    if e.response['Error']['Code'] == 'InvalidParameterValue':
        print('ERROR MESSAGE: {}'.format(e))"
if [ $? -eq 0 ]; then
    sudo reboot
fi
```

Install the following

```sh
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

sudo yum -y install jq gettext bash-completion moreutils

echo 'yq() {
  docker run --rm -i -v "${PWD}":/workdir mikefarah/yq "$@"
}' | tee -a ~/.bashrc && source ~/.bashrc

kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion

echo 'export LBC_VERSION="v2.4.1"' >>  ~/.bash_profile
echo 'export LBC_CHART_VERSION="1.4.1"' >>  ~/.bash_profile
.  ~/.bash_profile

aws cloud9 update-environment  --environment-id $C9_PID --managed-credentials-action DISABLE
rm -vf ${HOME}/.aws/credentials

export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion

for command in kubectl jq envsubst aws
  do
    which $command &>/dev/null && echo "OK: $command in path" || echo "$command NOT FOUND"
done
eksctl version
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
test -n "$S3_AWS_ACCESS_KEY_ID" && echo S3_AWS_ACCESS_KEY_ID is "$S3_AWS_ACCESS_KEY_ID" || echo S3_AWS_ACCESS_KEY_ID is not set
aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
aws eks update-kubeconfig --region us-east-2 --name eksworkshop-eksctl

```

3.	Generate a Personal Access Token in Github: 

Go to: Github > Settings > Developer Settings > Personal Access Tokens
Oauth Scope Access: Repo and Workflow

(Personal Access Tokens)[https://github.com/settings/tokens]

```
export GITHUB_USER=Github_user_name
export GITHUB_TOKEN=<YOur-access-token>
```

4. Create a AWS IAM user that has access to S3. 

(Create IAM User)[https://us-east-1.console.aws.amazon.com/iam/home#/users$new?step=review&accessKey&userNames=spinnaker-user-test&permissionType=policies&policies=arn:aws:iam::aws:policy%2FAmazonS3FullAccess]


Download the Access Key and Secret key and set it as environment variables. 
![Image](/images/IAM_USER_ACCESS-KEY.PNG)

```
export S3_AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
export S3_AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ID
```