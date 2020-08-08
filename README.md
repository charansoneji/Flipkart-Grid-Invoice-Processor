# Flipkart-Grid-2.0-Invoice-Processor
## Solution overview
<strong> Essentially we have created an API solution for FLIPKART which can be used in order to extract all the data from an invoice in a very efficient manner and it has been configured by using a DATABASE (DynamoDB) in order to maintain integrity of information and at the same time provide efficient and accurate results. The above solution is coupled along with a simple frontend and a python notebook which will obtain all the results and convert them into the required excel file which can then be provided to the customer. In this manner, essentially all the data which was present in the invoice is converted into an excel sheet. The solution is highly reliable because it makes use of AWS and it has been configured in a manner that it provides accurate and efficient result to the user. It provides the user with a frontend which is a major plus point as the user does not have to go through the hassle of opening the AWS console but can simply run a few CLI based commands. </strong>It is straightforward to invoke this API from AWS CLI or using Boto3 Python library and pass either a pointer to the document image stored in S3 or the raw image bytes to obtain results. However handling large volumes of documents this way becomes impractical for several reasons:<br>
<ul>
  <li>Making a synchronous call to query Textract API is not possible for multi-page PDF documents</li>
  <li>Synchronous call will exceed provisioned throughput if used for a large number of documents within a short period of time</li>
  <li>If multiple queries with same document is needed, triggering multiple Textract API invocation rapidly increases the cost</li>
  <li>Textract sends analysis results with rich metadata, but the strucutres of tables, forms and texts are not immediately apparent without some post-processing.</li>
</ul><br>
In Textract Enhancer solution, following approaches are used to provide for a more robust end to end solution.<br>
<ul>
  <li>Lambda functions triggered by document upload to specific S3 bucket to submit document analysis and text detection jobs to Textract</li>
  <li>API Gateway methods to trigger Textract job submission on-demand</li>
  <li>Asynchronous API calls to start Document analysis and Text detection, with unique request token to prevent duplicate submissions</li>
  <li>Use of SNS topics to get notified on completion of Textract jobs</li>
  <li>Automatically triggered post processing Lambda functions to extract actual tables, forms and lines of text, stored in S3 for future querying</li>
  <li>Job status and metadata tracked in DynamoDB table, allowing for troubleshooting and easy querying of results</li>
  <li>API Gateway methods to retrieve results anytime without having to use Textract</li>
</ul><br>
 </div>
  
Follow the below given steps in order to understand how to install the entire infrastrcture of this solution of invoice processing.<br>
If you are <strong>`testing`</strong> the given project, then head over to <strong>`SECTION B`</strong> but if you are looking at<br>
<strong>`installing the given project`</strong>, head over to <strong>`SECTION A`</strong>.<br>
## SECTION A - Installing the project individually (on a seperate machine and a seperate AWS account)<br>
<strong> Step 1: Creating an AWS account would be the first and most important step of this step.</strong><br>
One of the important things to note as a newly signed user is about the <a href="https://aws.amazon.com/free">free tier</a> which you get when you sign up as a new user with AWS. You can read about it in the hyperlink mentioned above. Click <a href="https://aws.amazon.com/aispl/registration-confirmation">here</a> in order to register as a new AWS user. You can read about the free tier on the AWS page or using the link provided above.<br>

<strong> Step 2: Open your AWS management console and search for the service called `Cloud Formation`. Click on `Create new Stack` and select the option which says, use a   predefined template. Now I have already defined the template which is available in this repository in `AWS STACK template` and is present as a JSON file. Copy the entire JSON file or attach it to Cloud Formation and wait for Cloud Formation to create the entire stack of resources that would be needed for the project.</strong><br>
Make sure to copy the entire JSON file and add the name of your required buckets or roles which are being defined which can be edited in the JSON.<br> 
###### Note: I have created the entire template based on the architecture that I had planned. You can adjust the JSON according to your requirement. The JSON is a very easy representation the architecture planned by me. It creates the roles required along with the database and the S3 bucket which would be needed. 
<strong> Step 3: Cross check if all the resources have been successfully deployed by refreshing the Cloud Formation page and looking for the `Stack Created` option.</strong><br>
This is an important step and if any error occurs during this stage, an immediate rollback shall occur which would mean that the resources that have been created are being destroyed. At this stage, you would have to check your IAM policies and make sure that the user has sufficient access. You can click <a href="https://aws.amazon.com/iam/">here</a> to understand more about the IAM policies.<br>
<strong> Step 4: You need to modify your CORS configuration in your S3 bucket. So head over to the S3 console and add the following configuration to your CORS config..</strong><br>
You need to 
