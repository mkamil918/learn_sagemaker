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
Install Docker desktop, sign up, and learn how to create images and run docker containers. Creating Dockerized solutions or applications is a good practice anyway, in case you plan on deploying it on a Kubernetes cluster. For this tutorial, we will use Docker to create and run the model in the Sagemaker console. 

#### 5: 








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







## Docker commands for ECR

Step 1: Hopefully you have added docker to path by now. Use the code below to authenticate docker with your ECR registry. The <your-region> parameter is determined when you do aws configure in the CLI, and <your-account-id> can be obtained by clicking on your username in the top right of the AWS homepage. 

```
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com
```













