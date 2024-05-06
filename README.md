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

#### 5: WSL (Windows Subsystem for Linux):

You need this to run Docker, but the Docker setup usually takes care of this for you. In case Docker does not install this automatically, check out the link [here](https://learn.microsoft.com/en-us/windows/wsl/install).

WSL is also useful in case you want to run Linux VMs on your machine, but for that you need to install Ubuntu as well, but that is outside the scope of this tutorial. 


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


## Create ECR Repo

Go to the ECR console [here](https://console.aws.amazon.com/ecr/)

Click on create repository, then add your respository name, and click on create. This repo will be used to store the docker image for the inference.py script. 


## Docker commands for ECR

Step 1: Hopefully you have added docker to path by now. Use the code below to authenticate docker with your ECR registry. The <your-region> parameter is determined when you do aws configure in the CLI, and <your-account-id> can be obtained by clicking on your username in the top right of the AWS homepage. 

You need to make sure to get the correct region; you can check it from the ECR repositories page, under URI. 
```
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com
```


## Train Model

Next, we train our model in python on our local machine using tensorflow. Below is the code to train the model. 

```
import tensorflow as tf

# Generate some example data
X_train = tf.random.normal(shape=(1000, 10))
y_train = tf.random.normal(shape=(1000, 1))

# Define a simple sequential model
model = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu', input_shape=(10,)),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dense(1)
])

# Compile the model
model.compile(optimizer='adam',
              loss='mean_squared_error',
              metrics=['mean_squared_error'])

# Train the model
model.fit(X_train, y_train, epochs=10, batch_size=32)

# Save the model
tf.keras.models.save_model(model, 'saved_model.keras')
```
You can access the script for the model training in the model_training folder.

After the model training has completed, run the following command in the CLI to push the saved model to the s3 bucket. 

```
aws s3 cp saved_model.keras s3://<your-bucket-name>/saved_model.keras
```

Next, create a script called inference.py with the following content:

```
import os
import json
import tensorflow as tf

# Define the model file name
MODEL_FILE = 'saved_model.keras'

# Define the model_fn function to load the model
def model_fn(model_dir):
    """
    Load the TensorFlow/Keras model.
    
    Parameters:
    - model_dir: Path to the directory containing the model artifacts
    
    Returns:
    - model: Loaded TensorFlow/Keras model
    """
    model_path = os.path.join(model_dir, MODEL_FILE)
    model = tf.keras.models.load_model(model_path)  # Load the model from the specified path
    return model

# Define the predict_fn function to handle input data and generate predictions
def predict_fn(input_data, model):
    """
    Generate predictions from the input data using the loaded model.
    
    Parameters:
    - input_data: Input data for prediction
    - model: Loaded TensorFlow/Keras model
    
    Returns:
    - predictions: Predicted output
    """
    # Perform inference
    predictions = model.predict(input_data)
    
    return predictions.tolist()  # Convert the predictions array to a list
```

Next, you need to containerize the inference.py script.


## Push inference.py to the ECR using Docker

Create a text file called dockerfile with no extension. Put the following content in it:

```
FROM tensorflow/tensorflow:latest

COPY inference.py inference.py

ENV SAGEMAKER_PROGRAM inference.py

ENTRYPOINT ["python", "inference.py"]
```

Next, run the following command in the terminal to build the docker image.

```
docker build -t <your-image-name> .
```

Next, you need to tag the Docker Image with the Amazon ECR registry path. Run the following command:
```
docker tag <your-image-name> <account_id>.dkr.ecr.<your-region>.amazonaws.com/<your-ecr-repo>:<tag>
```

Finally, use the docker push command to push the image to the ECR repo. 

```
docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/<your-ecr-repo>:<tag>
```

## Create role in Sagemaker:

The next step is to create the model in Sagemaker using the docker image we just pushed into ECR. For this, go into the IAM console, select roles, and create a new role. Under the Trusted entity type tab, select AWS service, and in usecase, select Sagemaker, and click on next on the bottom. You will see a tab called Permission policies, and under policy name you will see AmazonSageMakerFullAccess; click on next, assign a name to this role, and click on create role. I used aws-sagemaker-role, but you can use anything you like. 

## Give S3 permissions to Sagemaker role:

If you recall, the model was stored in S3, so we need to give the Sagemaker role full access to S3 as well. 

Open the IAM console, click on roles, select the role you created previously, click on add permissions, then go to attach policies, and look for AmazonS3FullAccess, and give this policy to the created role. 

## Create a SageMaker Model

In the SageMaker console or using the AWS SDK, you'll create a SageMaker Model. This involves specifying the container image in ECR, IAM roles, and other configurations. Run the following code in python:

```
import boto3

# Initialize SageMaker client
sagemaker_client = boto3.client('sagemaker')

# Specify the location of your container in ECR
container = {
    'Image': '<your-account-id>.dkr.ecr.<your-region>.amazonaws.com/<your-ECR-repo>:<image-tag>',

    'ModelDataUrl': 's3://<your-bucket-name>/saved_model.keras',  
    'Environment': {
        'key': 'value'  # Optional environment variables for your container
    }
}

# Specify the IAM role ARN
execution_role_arn = 'arn:aws:iam::<account_id>:role/<role_name>'

# Create the SageMaker Model
sagemaker_client.create_model(
    ModelName='modeltest',  # Name for your SageMaker Model
    ExecutionRoleArn=execution_role_arn,  # IAM role ARN
    PrimaryContainer=container  # Container configuration
)
```
You should see the following responses if the code is executed correctly:


ModelArn: This is the Amazon Resource Name (ARN) of the SageMaker Model that was created. The ARN uniquely identifies the model within AWS. You can use this ARN to reference the model in other AWS services or API calls.

ResponseMetadata: This section provides metadata about the API response:

    RequestId: A unique identifier for the request. This can be helpful for troubleshooting or tracking purposes.
    
    HTTPStatusCode: The HTTP status code of the response. A status code of 200 indicates that the request was successful.
    
    HTTPHeaders: Additional information about the HTTP response, including the content type, content length, and date.
    
    RetryAttempts: The number of retry attempts made by the AWS SDK.
 
## Deploy Model to Endpoint:

To deploy a SageMaker model to an endpoint using the AWS SDK, you can use the create_endpoint method provided by the SageMaker client. Run the following code in python.

```
endpoint_config_name = 'newendptcfg'  # Specify the endpoint configuration name
model_name = 'modeltest'  # Specify the model name
instance_type = 'ml.m4.xlarge'  # Specify the instance type
initial_instance_count = 1  # Specify the initial instance count

# Create endpoint configuration
sagemaker_client.create_endpoint_config(
    EndpointConfigName=endpoint_config_name,
    ProductionVariants=[{
        'InstanceType': instance_type,
        'InitialInstanceCount': initial_instance_count,
        'ModelName': model_name,
        'VariantName': 'AllTraffic'
    }]
)

# Create endpoint
endpoint_name = 'mlendpointnew'  # Specify the endpoint name
sagemaker_client.create_endpoint(
    EndpointName=endpoint_name,
    EndpointConfigName=endpoint_config_name
)

```

## Making inference requests on the model:

Now, you can start making inference requests, or "testing" the model against real world data. 

Run the following python code:

```

```






















