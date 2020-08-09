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
<strong> Step 4: You need to modify your CORS configuration in your S3 bucket. So head over to the S3 console and add the following configuration to your CORS config.</strong><br>
You need to head on to the s3 bucket and in "Permissions", click on CORS configuration and copy paste the below given config:<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
   <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedHeader>*</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```
<strong> Step 5: Testing the API gateway</strong><br>
You can test the API by heading over to the AWS API Gateway and by clicking on the API created using the url and making use of any of the below given links:<br>
<ul>
  <li>Textract Job start by API Invocation: If you already have documents present in bucket, or not the owner of the bucket, you can still trigger the same workflow as above, by sending a request to Rest API method as follows: https://deployment-id.execute-api.us-east-1.amazonaws.com/demo/submittextanalysisjob?Bucket=your-bucket-name&Document=your-document-key You can find the deployment-id of the API from the stack output.</li><br>
 <li>Textract result retrieval via Rest API: If the initial submission goes well, and does not exceed provisioned throughput for maximum number of trials, result will be ready and post-processed within few seconds to minutes. At that point, the document analysis result can be retrieved by invoking Rest API method as follows: https://deployment-id.execute-api.us-east-1.amazonaws.com/demo/retrievedocumentanalysisresult?Bucket=your-bucket-name&Document=your-document-key&ResultType=ALL|TABLE|FORM. Similarly text detection result can be obtained by invoking Rest API method as follows: https://deployment-id.execute-api.us-east-1.amazonaws.com/demo/retrievetextdetectionresult?Bucket=your-bucket-name&Document=your-document-key You can find the deployment-id of the API from the stack output. In both cases, the API response will contain a list of files on S3 bucket where the results are stored for future use. You can also download and open the result files, either to inspect the contents manually, or to feed in to some downstream application/processes, as needed. </li>
 </ul>
 <div>
<strong> Step 6: Clone the Fronend in order to upload the invoice to s3 via an interface</strong><br>
Click on the sub-repository named "Frontend" or click <a href="https://github.com/charansoneji/Flipkart-Grid-Invoice-Processor/tree/master/Frontend">here</a>.Clone the repo and make sure to install packages `express`, `aws-sdk` and `jade`. You can then run the command `node app.js` but you can find further instructions in the README given in that folder. <br>
</div>
