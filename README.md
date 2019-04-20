# SageMaker Deployment Project

You can look at the output using this link https://sand47.github.io/review-classification/, <br>
The notebook and Python files provided here, once completed, result in a simple web app which interacts with a deployed recurrent neural network performing sentiment analysis on movie reviews. 

# General Outline
Recall the general outline for SageMaker projects using a notebook instance.
<br>
1.Download or otherwise retrieve the data.
2.Process / Prepare the data.<br>
3.Upload the processed data to S3.<br>
4.Train a chosen model.<br>
5.Test the trained model (typically using a batch transform job).<br>
6.Deploy the trained model.<br>
7.Use the deployed model.<br>

For this project, you will be following the steps in the general outline with some modifications.<br>

First, you will not be testing the model in its own step. You will still be testing the model, however, you will do it by deploying your model and then using the deployed model by sending the test data to it. One of the reasons for doing this is so that you can make sure that your deployed model is working correctly before moving forward.<br>

In addition, you will deploy and use your trained model a second time. In the second iteration you will customize the way that your trained model is deployed by including some of your own code. In addition, your newly deployed model will be used in the sentiment analysis web app.<br>

# Deploy the model for the web app
<br>

After training the model and saving the endpoint we need create a path to access it through API gateway and Lambda functions. <br>

# Setting up a Lambda function<br>
The first thing we are going to do is set up a Lambda function. This Lambda function will be executed whenever our public API has data sent to it. When it is executed it will receive the data, perform any sort of processing that is required, send the data (the review) to the SageMaker endpoint we've created and then return the result.
<br>
## Part A: Create an IAM Role for the Lambda function<br>
Since we want the Lambda function to call a SageMaker endpoint, we need to make sure that it has permission to do so. To do this, we will construct a role that we can later give the Lambda function.
<br>
Using the AWS Console, navigate to the IAM page and click on Roles. Then, click on Create role. Make sure that the AWS service is the type of trusted entity selected and choose Lambda as the service that will use this role, then click Next: Permissions.
<br>
In the search box type sagemaker and select the check box next to the AmazonSageMakerFullAccess policy. Then, click on Next: Review.
<br>
Lastly, give this role a name. Make sure you use a name that you will remember later on, for example LambdaSageMakerRole. Then, click on Create role.
<br>
## Part B: Create a Lambda function<br>
Now it is time to actually create the Lambda function.
<br>
Using the AWS Console, navigate to the AWS Lambda page and click on Create a function. When you get to the next page, make sure that Author from scratch is selected. Now, name your Lambda function, using a name that you will remember later on, for example sentiment_analysis_func. Make sure that the Python 3.6 runtime is selected and then choose the role that you created in the previous part. Then, click on Create Function.
