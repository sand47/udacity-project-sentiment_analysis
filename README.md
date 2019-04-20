# SageMaker Deployment Project

Deployed webpage https://sand47.github.io/review-classification/ <br><br>
The notebook and Python files provided here, once completed, result in a simple web app which interacts with a deployed recurrent neural network performing sentiment analysis on movie reviews. 

# General Outline
The general outline for SageMaker projects using a notebook instance.
<br><br>
1.Download or otherwise retrieve the data.<br>
2.Process / Prepare the data.<br>
3.Upload the processed data to S3.<br>
4.Train a chosen model.<br>
5.Test the trained model (typically using a batch transform job).<br>
6.Deploy the trained model.<br>
7.Use the deployed model.<br>

For this project, you will be following the steps in the general outline with some modifications.<br>
<br>
First, you will not be testing the model in its own step. You will still be testing the model, however, you will do it by deploying your model and then using the deployed model by sending the test data to it. One of the reasons for doing this is so that you can make sure that your deployed model is working correctly before moving forward.<br>
<br>
In addition, you will deploy and use your trained model a second time. In the second iteration you will customize the way that your trained model is deployed by including some of your own code. In addition, your newly deployed model will be used in the sentiment analysis web app.<br>

# Prerequisite 
<br>
Create an AWS account https://aws.amazon.com/ and make sure you know python if not you can learn from this link https://www.youtube.com/watch?v=oVp1vrfL_w4&list=PLQVvvaa0QuDe8XSftW-RAxdo6OmaeL85M
<br>

# Deploy the model for the web app
<br>
After training the model and saving the endpoint we need create a path to access it through API gateway and Lambda functions. <br>
<br>

# Setting up a Lambda function<br>
The first thing we are going to do is set up a Lambda function. This Lambda function will be executed whenever our public API has data sent to it. When it is executed it will receive the data, perform any sort of processing that is required, send the data (the review) to the SageMaker endpoint we've created and then return the result.
<br>

## Part A: Create an IAM Role for the Lambda function<br>
Since we want the Lambda function to call a SageMaker endpoint, we need to make sure that it has permission to do so. To do this, we will construct a role that we can later give the Lambda function.
<br><br>
Using the AWS Console, navigate to the IAM page and click on **Roles**. Then, click on Create role. Make sure that the AWS service is the type of trusted entity selected and choose Lambda as the service that will use this role, then click Next: Permissions.
<br><br>
In the search box type **sagemaker** and select the check box next to the **AmazonSageMakerFullAccess** policy. Then, click on Next: Review.
<br><br>
Lastly, give this role a name. Make sure you use a name that you will remember later on, for example LambdaSageMakerRole. Then, click on Create role.
<br><br>
## Part B: Create a Lambda function<br>
Now it is time to actually create the Lambda function.
<br><br>
Using the AWS Console, navigate to the AWS Lambda page and click on **Create a function**. When you get to the next page, make sure that Author from scratch is selected. Now, name your Lambda function, using a name that you will remember later on, for example sentiment_analysis_func. Make sure that the **Python 3.6** runtime is selected and then choose the role that you created in the previous part. Then, click on Create Function.
<br>
On the next page you will see some information about the Lambda function you've just created. If you scroll down you should see an editor in which you can write the code that will be executed when your Lambda function is triggered. In our example, we will use the code below.
<br>
   
    import boto3
    def lambda_handler(event, context): 
        # The SageMaker runtime is what allows us to invoke the endpoint that we've created.
        runtime = boto3.Session().client('sagemaker-runtime')
    # Now we use the SageMaker runtime to invoke our endpoint, sending the review we were given
        response = runtime.invoke_endpoint(EndpointName = '**ENDPOINT NAME HERE**',    # The name of the endpoint we created
                                       ContentType = 'text/plain',                 # The data format that is expected
                                       Body = event['body'])                       # The actual review

    # The response is an HTTP response whose body contains the result of our inference
        result = response['Body'].read().decode('utf-8')

        return {
            'statusCode' : 200,
            'headers' : { 'Content-Type' : 'text/plain', 'Access-Control-Allow-Origin' : '*' },
            'body' : result
                }
<br>
Once you have copy and pasted the code above into the Lambda code editor, replace the **ENDPOINT NAME HERE** portion with the name of the endpoint that we deployed earlier.

# Setting up API Gateway
Now that our Lambda function is set up, it is time to create a new API using API Gateway that will trigger the Lambda function we have just created.
<br><br>
Using AWS Console, navigate to Amazon API Gateway and then click on Get started.
<br><br>
On the next page, make sure that New API is selected and give the new api a name, for example, sentiment_analysis_api. Then, click on Create API.
<br><br>
Now we have created an API, however it doesn't currently do anything. What we want it to do is to trigger the Lambda function that we created earlier.
<br><br>
Select the **Actions dropdown** menu and click **Create Method**. A new blank method will be created, select its dropdown menu and select **POST**, then click on the check mark beside it.
<br><br>
For the integration point, make sure that **Lambda Function is selected and click on the Use Lambda Proxy integration**. This option makes sure that the data that is sent to the API is then sent directly to the Lambda function with no processing. It also means that the return value must be a proper response object as it will also not be processed by API Gateway.
<br><br>
Type the name of the Lambda function you created earlier into the Lambda Function text entry box and then click on Save. Click on OK in the pop-up box that then appears, giving permission to API Gateway to invoke the Lambda function you created.
<br><br>
The last step in creating the API Gateway is to select the Actions dropdown and click on Deploy API. You will need to create a new Deployment stage and name it anything you like, for example prod.
<br><br>
You have now successfully set up a public API to access your SageMaker model. Make sure to **copy or write down the URL** provided to invoke your newly created public API as this will be needed in the next step. This URL can be found at the top of the page, highlighted in blue next to the text Invoke URL.
<br><br>

# Last step
Copy the above API url and paste it in the index.html page to access the endpoint. 
<br><br>
If you'd like to go further, you can host this html file anywhere you'd like, for example using github or hosting a static site on Amazon's S3. Once you have done this you can share the link with anyone you'd like and have them play with it too!
