# api-gateway-lambda-aws
Test API Gateway and Lambda

Test API Gateway and Lambda

Version: 0.1
Release Date: February 2016




Table of Contents

Table of Contents
Version History
Contributors
Reviewers
About the document
Resolution
Explain which HTTP method you used in 1., and why
Which language did you choose to do the lambda function, and why?
If you wanted to support different output formats, what would you do?











































Version History
Version
Release Date
Description of Changes
0.1
February 2016
First Draft









Contributors

S.No.
Name
1.
Patrick Rodies

Reviewers

Role
Approver
✓
Tech
Patrick Rodies






About the document
This document will describe all steps to resolve the Business Systems Developer challenge.
Resolution
Explain which HTTP method you used in 1., and why
Created a HTTP GET method in order to pass an argument called answer.  In order to do so I have created a API Gateway using AWS Console. This API use a resource called challenge and a GET method for this resource called answer. The method has a Lambda Function as integration type. 



4down vote
In order to pass the parameter to the lambda function I needed to create a mapping between the API Gateway request and the lambda function. The mapping is done in the Integration Request -> Mapping templates section of the selected API Gateway resource.
The template used was:
{ "querystring" : "#foreach($key in $input.params().querystring.keySet())#if($foreach.index > 0)&#end$util.urlEncode($key)=$util.urlEncode($input.params().querystring.get($key))#end", "body" : $input.json('$') }
Note: click on the check symbol to save the template. You can test your changes with the "test" button in your resource. But in order to test querystring parameters in the AWS console you will need to define the parameter names in the Method Request section of your resource.
Then in my lambda template I can do the following to get the query string parsed:
var query = require('querystring').parse(event.querystring) // access parameters with query['answer'] or query.answer

The lambda function and the API Gateway are attached to a specific user with limited privileges.
The Lambda code is as following
const http = require('http');
console.log('Loading event');
var resp;

exports.handler = function(event, context) {
    var query = require('querystring').parse(event.querystring);
    console.log('Query:', query);
    var md5Var = require('crypto').createHash('md5').update(query.answer).digest("hex");
    console.log('Answer:', query.answer);
    //get answer from url
    var options = {
        host: 'followthewhiterabbit.trustpilot.com',
        port: 80,
        path: '/sd/'+md5Var+'.json'
    };
    http.get(options, function(res) {
        resp = res.statusCode;
        console.log("Got response: " + resp);
        context.succeed({"Response":resp,"MD5":md5Var,"String":query});
        
    }).on('error', function(e) {
        console.log("Got error: " + e.message);
        context.done(null, 'FAILURE');
});
};
Which language did you choose to do the lambda function, and why?
	I choose node.js as I used it in the past for similar tasks.
If you wanted to support different output formats, what would you do?
In order to support outgoing XML you need to edit the method response. By default for each method you create the API Gateway console creates a 200 response with content-type 'application/json'. Edit this response to be content-type 'application/xml'.

You can check this implementation at https://b3ta9j9519.execute-api.eu-west-1.amazonaws.com/beta/challenge/?answer=nebuchadnezzar

By changing the parameter value you can check if the answer is correct:
Correct answer will return "Response":200
Wrong answer will return "Response":404



