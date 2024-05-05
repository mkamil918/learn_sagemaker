# learn_sagemaker
## A comprehensive guide to deploying ML models in AWS Sagemaker.

This is a multistep tutorial that is based on my own learning experience with the AWS Sagemaker tool. To get started with this tutorial, you will need an AWS account; the sign in process is fairly straightforward, and even though it requires a credit card, you will not be charged as long as you stay within the free tier, which is fairly extensive and forgiving. Additionally, AWS allows you to set up billing alerts to your email, which get triggered as soon as you cross the 0.01 USD threshold. 

### Requirements:

#### 0: Powershell
You need to be familiar with shell commands in Windows (as I am a Windows user, and this tutorial will use windows shell commands). However, if you uses a Linux based OS, you should be able to find the relevant shell commands for your OS. You should also add python to your path variable, in case you haven't already; this will come in handy when trying to run other services (such as docker or AWS CLI) via the CLI. 

#### 1: Python 
You need to have python set up on your local machine, along with the required libraries installed (pandas, numpy, sklearn, tensorflow).

#### 2: Knowledge of ML Algorithms:
Since this is a tutorial on deploying ML algorithms, you need to have at least a rudimentary understanding of the machine learning model building process, model training, inference, saving models, and model evaluation. 

#### 4: Docker: 
Install Docker desktop, sign up, and learn how to create images and run docker containers. Creating Dockerized solutions or applications is a good practice anyway, in case you plan on deploying it on a Kubernetes cluster.. For this tutorial, we will use Docker to create and run the model in the Sagemaker console. 


## Configure AWS:

This is the initial step to using the AWS CLI on your local machine. 

#### 1: Install AWS CLI:
Run this command in the terminal. 

```
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
To check if the AWS CLI is installed, run this command in powershell or terminal: 

```
aws --version
```

#### 2: Configure AWS CLI: 

Configure the AWS CLI with your user's access keys and regions. 

```
aws configure
```
This will prompt you to supply the access key, secret access key, region, and output format. For the region, I used us-east-1. In order to obtain the access key and secret access key, go to the IAM panel, click on users in the left navigation pane, and select the user you created prior. It should have an access key and secret access key in that tab. For output, I used json. 


## Configure S3: 

S3 (Amazon Simple Storage Service) is a cloud based object storage service, that can be used to store any kind of data (CSVs, images, etc). In our use, we will use S3 to store our model, after exporting it from our local environment using the save model command. 

### Create a policy:
In order to create an s3 bucket, you need to create a policy and assign that policy to your user. Open your text editor, and copy and paste this bit of information:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3BucketCreation",
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```
This policy allows the user to create an S3 bucket. 

Next, go to the IAM console, click on policies on the left navigation bar, click on create policy, then click on JSON, paste the text created above, and click on next. In this window, add the name (required) and description (optional), and click on create policy. 

Repeat this step for the following text, and add it to a new policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::mkmltestbucket/*"
        }
    ]
}
```
This policy allows the user to store objects in the S3 bucket after its creation. 

### Assign Policy to User:

To assign the created policies to the user, go to the IAM console, click on users, select your user, click on "Add permissions". A dropdown will appear, from that menu, select "add permissions". In the new window, you will see three options: "add user to group", "copy permissions", "attach policies directly". In this case, we will use the third option "attach policies directly". A number of policies will now appear on the bottom, you need to search the following policies and attach those to your user:

1: The create s3 bucket policy created in the previous step.
2: The put object in s3 bucket policy created in the previous step.
3: AmazonEC2ContainerRegistryFullAccess

## Create S3 Bucket:

Once the user has the required permissions, you need to create the S3 bucket that will store your model. 

Run this command in the CLI:

```
aws s3 mb s3://mltestbucket --region us-east-1
```
Your bucket name can be anything, barring some exceptions. You can check the [documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html) here. 



## Docker commands for ECR

Step 1: Hopefully you have added docker to path by now. Use the code below to authenticate docker with your ECR registry. The <your-region> parameter is determined when you do aws configure in the CLI, and <your-account-id> can be obtained by clicking on your username in the top right of the AWS homepage. 

```
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com
```



















