# AWS-Cloud-Project---Lambda-and-S3
# Integrate Amazon S3 with Your Django-Based File Handler Application Using AWS Lambda for File Uploads

In this tutorial, we’ll walk through the steps to integrate Amazon S3 with your Django-based filehandler project using AWS Lambda.

I have developed a simple application using Python Django that allows users to upload files, which are then stored in a database. Currently, this application is hosted in my local environment, and the files are saved in a PostgreSQL database. However, instead of storing these files in the database, I want to upload them to Amazon S3 using AWS Lambda.  

You can find the application in the GitHub repository below:  

```
https://github.com/luciyaroshni/File-Handler-Django
```
What’s happening behind the scenes? AWS Lambda serves as the serverless function that processes the file uploads. When a file is uploaded through your Django application, the file data is sent to an API Gateway, which triggers the Lambda function. The Lambda function then processes the file and uploads it to an S3 bucket.    

![image](https://github.com/user-attachments/assets/39bc0ff5-fb13-49ad-9cdf-f37d645deecb)  

### Step 1: Create S3 Bucket

First, we need to create an S3 bucket, which will serve as the storage location for our files. Before proceeding, ensure that you are in the correct region. To create the S3 bucket, login to your AWS Management Console and search for S3 in the search bar.

Click on Create bucket.

![image](https://github.com/user-attachments/assets/51780e04-57d3-45a2-9a1a-d8287d8b4998)

Enter a unique name for the bucket. You can choose to disable object ownership if desired.

Why do we have to disable the object ownership? If you disable object ownership, it means that the person or system that uploads a file will be the one who controls it. In short, disabling object ownership means that whoever uploads a file keeps control of that file, which can be helpful when many people or systems are using the same bucket.

![image](https://github.com/user-attachments/assets/bd3bb393-0dc7-42ad-bca3-7a6ea9ca980f)

Uncheck Block all public access.

![image](https://github.com/user-attachments/assets/3e4e292c-1be6-4dcf-90c6-2dbd8f854460)

Enable Bucket Versioning. This feature offers several benefits:

§ Data Protection: It guards against accidental overwrites and deletions by keeping previous versions of your files.

§ Change Tracking: It lets you monitor changes made to your files over time.

§ Error Recovery: If files are mistakenly deleted or overwritten, you can easily restore earlier versions.

![image](https://github.com/user-attachments/assets/daa8c6e4-8c38-47ae-aaa4-c8d043ad9fc1)

In the Default Encryption section, you can choose the encryption type. Here, I have chosen SSE-S3, which stands for Server-Side Encryption with Amazon S3-Managed Keys. This means that Amazon S3 handles the encryption for you. When you upload files, S3 automatically encrypts them before storing them, helping to protect your data at rest.

![image](https://github.com/user-attachments/assets/9385f0ae-11e3-4c7a-83da-c3b3e8b8822c)

### Step 2: Set up the AWS Lambda Function

Navigate to the AWS Lambda service in your AWS Management Console, and click on Create Function.

![image](https://github.com/user-attachments/assets/2033768f-a441-4f05-86b4-b79ec7530b27)

Select the Author from scratch option to start with a blank function.

Choosing “Author from scratch” allows you to create a Lambda function from the ground up, giving you full control over the function’s code, configuration, and environment. This option is ideal when you want to implement custom logic or when there isn’t an existing blueprint or template that fits your specific needs. It essentially provides a blank canvas for developing a Lambda function tailored to your application’s requirements.

Then, give your function a meaningful name, choose the appropriate runtime for your code, and select the architecture that suits your needs.

![image](https://github.com/user-attachments/assets/a5a073de-6493-46a3-8d59-318def9f831e)

Select Create a new role with basic Lambda permissions.

This option automatically generates a new IAM role with minimal permissions necessary for your Lambda function to run, making it easier to get started without manually configuring permissions.

Then click on Create Function.

![image](https://github.com/user-attachments/assets/c9ea2631-a5d3-49fa-b3c4-d5f8c8daf991)

Navigate to IAM roles in the AWS Management Console.

Select the role that was automatically created for your Lambda function.

Click Add permissions and then choose Create inline policy.

In the policy editor, provide the policy as below and create policy.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::filehandler-bucket/*"
    }
  ]
}
```

### Step 3: S3 Bucket Policy

This policy applies to your S3 bucket itself. It is optional to set a bucket policy, but if you need to control who can upload or download files at the bucket level, this is the place to do it.

To do this, navigate to your S3 bucket.

![image](https://github.com/user-attachments/assets/9676e6d4-6cfd-4ded-a779-e0c3e669e780)

Navigate to the Permissions tab of your S3 bucket.

![image](https://github.com/user-attachments/assets/443a0b8f-6fcb-4abf-8067-a48ee705be3e)

Scroll down to the Bucket policy section.

![image](https://github.com/user-attachments/assets/e4f9b2ab-3939-4b01-8dee-ed0136775caf)

If there is no existing policy, you will see an empty editor. If a policy already exists, you can choose to edit it or replace it with a new one.

I have generated the policy and provided the details.

![image](https://github.com/user-attachments/assets/6a03f286-9733-464e-9b4e-3d1953f3af56)

### Step 4: Update Lambda Function

Next, we need to update the Lambda function. This function acts as the backend processor, managing the file upload from your Django application and storing the file in the S3 bucket.

To deploy this code, navigate to your Lambda function in the AWS console, open the function editor, and replace the existing code with the updated version.

```
import json
import boto3
import base64

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    bucket_name = 'filehandler-bucket'
    
    file_data = base64.b64decode(event['body'])
    file_name = event['headers']['file-name']
    
    s3.put_object(Bucket=bucket_name, Key=file_name, Body=file_data)
    
    return {
        'statusCode': 200,
        'body': json.dumps('File uploaded successfully!')
    }
```

Here’s an explanation of the Lambda function I used to handle and store data in S3:

json: Used for handling JSON data.

boto3: AWS SDK for Python, which allows interaction with S3.

base64: For encoding and decoding base64 data.

lambda_handler: The entry point for the Lambda function. This function is triggered by the API Gateway when a request is made.

boto3.client(‘s3’): Initializes an S3 client that allows you to interact with your S3 bucket.

bucket_name : The name of your S3 bucket where files will be stored.

file_data: Decodes the base64 encoded file data received in the event body.

file_name: Retrieves the file name from the event headers.

s3.put_object: Uploads the file to the specified S3 bucket with the given file name.

statusCode: HTTP status code indicating the request was successful.

body: JSON response body confirming that the file was uploaded successfully.

After replacing the code, click on Deploy to update the Lambda function with the new changes.

![image](https://github.com/user-attachments/assets/ca2f2919-8af9-4898-acb1-605d8763c75e)

### Step 5: Setup an API Gateway

Next, we need to set up an API Gateway to trigger the Lambda function.

For doing that go to the API Gateway service in the AWS console.

![image](https://github.com/user-attachments/assets/65a08384-9de6-4419-967d-1a55e407f90a)

I choose REST API(This is a good choice for general-purpose APIs, and it provides a lot of flexibility and features.)

Click on Build.

![image](https://github.com/user-attachments/assets/5ba5db0f-53e7-4f78-9014-b4e5ba9a5f54)

Choose New API

Provide API name and Description

And chose endpoint type — regional (If your primary audience is within a specific region and you need lower latency within that region.)

![image](https://github.com/user-attachments/assets/76036cc0-0474-4347-8aa0-69a7cefc5d02)

Click on Create Resource.

![image](https://github.com/user-attachments/assets/6df63559-9895-4bac-b9f4-05f9fc17277d)

Provide Resource name.

Click on Create resource.

![image](https://github.com/user-attachments/assets/b9d22b3e-45f8-48d5-b43b-687e2385f9e2)

Select the `/upload` resource you have created. Then, click on Create method to add a new method to this resource.

![image](https://github.com/user-attachments/assets/c4a55e63-495b-4377-8079-ee07728d3b80)

Choose POST as the method type.

For the Integration Type, select Lambda Function.

Make sure to enable Lambda Proxy Integration to allow API Gateway to handle the request and pass it directly to your Lambda function.

![image](https://github.com/user-attachments/assets/5979391a-98a0-4268-bab5-feb8473d56cf)

Click on Create Method.

Once you have created the method and configured the integration, it’s time to deploy your API. To do this, click on Deploy API option.

![image](https://github.com/user-attachments/assets/c389a716-845e-4512-9f8a-a4e808a06138)

Choose the deployment stage from the drop-down menu. If no stage exists yet, click on Create new stage and provide a name for the new stage.

![image](https://github.com/user-attachments/assets/b2decb18-59fc-49f9-b181-5ad4185422ed)

Note the invoke URL.

![image](https://github.com/user-attachments/assets/4516f47b-f9dd-452e-b5fc-cad47903d5ad)

### Step 6: Integrate with Django

To integrate your Django application with the AWS API Gateway, you need to send HTTP requests to the API endpoint. That is why you have to install the `requests` library. This library will help you make HTTP requests to the API Gateway endpoint that triggers your AWS Lambda function.

Open your terminal or command prompt where your Django project is located.

Activate your virtual environment if you are using one.

Install the requests library by running the following command:

```
pip install requests
```

Now that you have the request library installed, you need to update your views.py to use it for sending HTTP requests.

Below is how it looks like:

```
from django.shortcuts import render
from django.http import HttpResponse
import requests
import base64

def upload_file_view(request):
    if request.method == 'POST' and request.FILES.get('file'):
        file = request.FILES['file']
        
        # Encode file content to base64
        encoded_file_data = base64.b64encode(file.read()).decode('utf-8')
        
        # URL of the API Gateway endpoint
        url = 'https://your-api-id.execute-api.your-region.amazonaws.com/dev/upload'
        
        # Headers for the request
        headers = {
            'Content-Type': 'application/octet-stream',
            'file-name': file.name
        }
        
        # Send the base64 encoded file data
        response = requests.post(url, data=encoded_file_data, headers=headers)
        
        # Check the response status
        if response.status_code == 200:
            return HttpResponse('File uploaded successfully!')
        else:
            return HttpResponse(f'File upload failed! Status code: {response.status_code}', status=response.status_code)
    return render(request, 'FileHandler1.html')
```

To make sure your Django application correctly routes requests to your upload_file_view function, you need to map it in your urls.py file.

Open your urls.py file in the app where you have your upload_file_view function. This file is typically found in the same directory as your views.py file.

Add a new URL pattern that maps to the upload_file_view function. Here’s an example of how to do it:

```
from django.urls import path
from . import views

urlpatterns = [
    path('upload/', views.upload_file_view, name='upload_file'),
]
```

Ensure that your app’s URLs are included in your project’s main urls.py file. This file is usually located in your project’s root directory (the directory with settings.py).

Open the main urls.py file and make sure it includes a reference to your app’s URLs. For example:

```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),          # Admin interface
    path('file/', include('file.urls'))       # Includes URL patterns from the 'file' app
]
```

By following these steps, you ensure that requests to the /upload/ URL are correctly routed to the upload_file_view function in your Django application. This setup allows your application to handle file uploads and interact with the AWS API Gateway as intended.

Now try to upload your files and see if it is working.

![image](https://github.com/user-attachments/assets/9ba2d6a1-97ab-4f7c-a43a-175fc077671b)

![image](https://github.com/user-attachments/assets/f0f67e4c-5e49-4777-8d8c-b54e640ba3f1)

![image](https://github.com/user-attachments/assets/90e95efe-9f36-49bc-86f3-633c65fb2670)

Let’s verify it on your S3 bucket.

![image](https://github.com/user-attachments/assets/0c25455e-d3c3-4f0c-9a4f-2981a6b50c7b)

As demonstrated, I successfully uploaded the file contents. I’d love to hear how it works out for you — feel free to share your results!
