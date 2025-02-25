# Configure a Lambda authorizer using the API Gateway console<a name="configure-api-gateway-lambda-authorization-with-console"></a>

 After you create the Lambda function and verify that it works, use the following steps to configure the API Gateway Lambda authorizer \(formerly known as the custom authorizer\) in the API Gateway console\. 

**To configure a Lambda authorizer using the API Gateway console**

1.  Sign in to the API Gateway console\. 

1.  Create a new or select an existing API and choose **Authorizers** under that API\. 

1.  Choose **Create New Authorizer**\. 

1.  For **Create Authorizer**, type an authorizer name in the **Name** input field\. 

1.  For **Type**, choose the **Lambda** option\. 

1.  For **Lambda Function**, choose a region and then choose an available Lambda authorizer function that's in your account\. 

1.  Leave **Lambda Invoke Role** blank to let the API Gateway console set a resource\-based policy\. The policy grants API Gateway permissions to invoke the authorizer Lambda function\. You can also choose to type the name of an IAM role to allow API Gateway to invoke the authorizer Lambda function\. For an example of such a role, see [Create an assumable IAM role](integrating-api-with-aws-services-lambda.md#api-as-lambda-proxy-setup-iam-role-policies)\. 

    If you choose to let the API Gateway console set the resource\-based policy, the **Add Permission to Lambda Function** dialog is displayed\. Choose **OK**\. After the Lambda authorization is created, you can test it with appropriate authorization token values to verify that it works as expected\. 

1.  For **Lambda Event Payload**, choose either **Token** for a `TOKEN` authorizer or **Request** for a `REQUEST` authorizer\. \(This is the same as setting the [type](https://docs.aws.amazon.com/apigateway/latest/api/API_Authorizer.html#type) property to `TOKEN` or `REQUEST`\.\) 

1. Depending on the choice of the previous step, do one of the following:

   1.  For the **Token** options, do the following: 
      + Type the name of a header in **Token Source**\. The API client must include a header of this name to send the authorization token to the Lambda authorizer\. 
      + Optionally, provide a RegEx statement in `Token Validation` input field\. API Gateway performs initial validation of the input token against this expression and invokes the authorizer upon successful validation\. This helps reduce chances of being charged for invalid tokens\.
      + For **Authorization Caching**, select or clear the **Enabled** option, depending on whether you want to cache the authorization policy generated by the authorizer or not\. When policy caching is enabled, you can choose to modify the **TTL** value\. Setting **TTL** to zero disables policy caching\. When policy caching is enabled, the header name specified in **Token Source** becomes the cache key\.
**Note**  
The default **TTL** value is 300 seconds\. The maximum value is 3600 seconds; this limit cannot be increased\.

   1. For the **Request** option, do the following:
      + For **Identity Sources**, type a request parameter name of a chosen parameter type\. Supported parameter types are `Header`, `Query String`, `Stage Variable`, and `Context`\. To add more identity sources, choose **Add Identity Source**\. 

        API Gateway uses the specified identity sources as the request authorizer caching key\. When caching is enabled, API Gateway calls the authorizer's Lambda function only after successfully verifying that all the specified identity sources are present at runtime\. If a specified identify source is missing, null, or empty, API Gateway returns a `401 Unauthorized` response without calling the authorizer Lambda function\. 

        When multiple identity sources are defined, they all used to derive the authorizer's cache key\. Changing any of the cache key parts causes the authorizer to discard the cached policy document and generate a new one\.
      + For **Authorization Caching**, select or deselect the **Enabled** option, depending on whether you want to cache the authorization policy generated by the authorizer or not\. When policy caching is enabled, you can choose to modify the **TTL** value from the default \(300\)\. Setting TTL=0 disables policy caching\.

        When caching is disabled, it is not necessary to specify an identity source\.
**Note**  
 To enable caching, your authorizer must return a policy that is applicable to all methods across an API\. To enforce method\-specific policy, you can set the TTL value to zero to disable policy caching for the API\. 

1. Choose **Create** to create the new Lambda authorizer for the chosen API\.

1.  After the authorizer is created for the API, you can optionally test invoking the authorizer before it is configured on a method\. 

    For the `TOKEN` authorizer, type a valid token in the **Identity token** input text field and the choose **Test**\. The token will be passed to the Lambda function as the header you specified in the **Identity token source** setting of the authorizer\. 

    For the `REQUEST` authorizer, type the valid request parameters corresponding to the specified identity sources and then choose **Test**\. 

    In addition to using the API Gateway console, you can use AWS CLI or an AWS SDK for API Gateway to test invoking an authorizer\. To do so using the AWS CLI, see [test\-invoke\-authorizer](https://docs.aws.amazon.com/cli/latest/reference/apigateway/test-invoke-authorizer.html)\. 
**Note**  
Test\-invoke for method executions test\-invoke for authorizers are independent processes\.   
To test invoking a method using the API Gateway console, see [Use the console to test a REST API method](how-to-test-method.md)\. To test invoking a method using the AWS CLI, see [test\-invoke\-method](https://docs.aws.amazon.com/cli/latest/reference/apigateway/test-invoke-method.html)\.   
To test invoking a method and a configured authorizer, deploy the API, and then use cURL or Postman to call the method, providing the required token or request parameters\.

 The next procedure shows how to configure an API method to use the Lambda authorizer\. 

**To configure an API method to use a Lambda authorizer**

1.  Go back to the API\. Create a new method or choose an existing method\. If necessary, create a new resource\. 

1.  In **Method Execution**, choose the **Method Request** link\. 

1.  Under **Settings**, expand the **Authorization** drop\-down list to select the Lambda authorizer you just created \(for example, **myTestApiAuthorizer**\), and then choose the check mark icon to save the choice\. 

1.  Optionally, while still on the **Method Request** page, choose **Add header** if you also want to pass the authorization token to the backend\. In **Name**, type a header name that matches the **Token Source** name you specified when you created the Lambda authorizer for the API\. Then, choose the check mark icon to save the settings\. This step does not apply to `REQUEST` authorizers\. 

1. Choose **Deploy API** to deploy the API to a stage\. Note the **Invoke URL** value\. You need it when calling the API\. For a `REQUEST` authorizer using stage variables, you must also define the required stage variables and specify their values while in **Stage Editor**\. 