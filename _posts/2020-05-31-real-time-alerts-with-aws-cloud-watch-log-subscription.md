---
title: Real time alerts with AWS cloud watch log subscription and SES - part1
slug: real-time-alerts-with-aws-cloud-watch-log-subscription
date_published: 2020-05-31T18:01:29.000Z
date_updated: 2020-05-31T18:10:30.000Z
tags: aws, node, cloudwatch, ses, lambda, alert
---

If you are working with lambda functions / batch processes in your AWS stack, chances are that you may find it difficult to track errors in the log. Lambda / batch invocation may write events in different log stream and at the end of day, you have a lot of stream if you need to scan and see if there was any unexpected event.

Thankfully, you can automate this process with Cloud watch and [SES](https://aws.amazon.com/ses/). 

For example, if you would like to get a notification every time there is an error logged in logs, you can create a cloud watch log subscription where you provide some keywords which you want to check and a lambda function which cloud watch invokes if it finds those keywords in any of the events. This invoked lambda can send an email to any email address you provide. With this flow in place, you get a real time alert whenever any error is logged.

Lets see some code.  

For aws resources, I'm using cloudformation template. If you want to directly create these resources, please refer to the links at the end of article.

### Creating Lambda function

    var zlib = require('zlib');
    var AWS = require('aws-sdk');
    var ses_client = new AWS.SES({ apiVersion: '2010-12-01' });
    
    const To = <email address>
    const From = <email address>
    
    exports.handler = function(input, context) {
        var payload = Buffer.from(input.awslogs.data, 'base64');
        zlib.gunzip(payload, function(e, result) {
            if (e) { 
                context.fail(e);
            } else {
                result = JSON.parse(result.toString('ascii'));
                console.log("Event Data:", JSON.stringify(result, null, 2));
                await send_mail(subject, JSON.stringify(result, null, 2),To,From);
                context.succeed();
            }
        });
    };
    
    var send_mail = async(subject, message,To,From) => {
        var params = {
            Destination: {
                ToAddresses: [
                    To,
                ]
            },
            Message: {
                Body: {
                    Text: {
                        Charset: "UTF-8",
                        Data: message
                    }
                },
                Subject: {
                    Charset: 'UTF-8',
                    Data: subject
                }
            },
            Source: From,
        };
    
        // Create the promise and SES service object
        var sendPromise = ses_client.sendEmail(params).promise();
    
        // Handle promise's fulfilled/rejected states
        await sendPromise.then(
            function(data) {
                console.log(JSON.stringify(data));
            }).catch(
            function(err) {
                console.error(err, err.stack);
            });
    };

index.js
    LogSubscriptionProcessor:
        Type: AWS::Lambda::Function
        Properties:
          Description: processes event from cloudwatch log subscription
          Code:
            S3Bucket: my-bucket
            S3Key: LogSubscriptionProcessor.zip
          FunctionName: Log_Subscription_Processor
          Handler: index.handler
          Runtime: nodejs10.x
          MemorySize: 2048
          Timeout: 120
          Role: !GetAtt MyRole.Arn   

### Giving cloud watch permission to invoke lambda

    AlertProessPermission:
        Type: AWS::Lambda::Permission
        Properties: 
          Action: lambda:InvokeFunction
          FunctionName: !GetAtt LogSubscriptionProcessor.Arn
          Principal: logs.us-west-2.amazonaws.com
          SourceAccount: !Sub ${AWS::AccountId}
          SourceArn: !Sub arn:aws:logs:us-west-2:${AWS::AccountId}:log-group:/aws/lambda/my-lambda:*

cloud formation template for giving permission to cloud watch to invoke lambda function
### Creating a subscription filter 

    SubscriptionFilter:
        Type: AWS::Logs::SubscriptionFilter
        Properties: 
          DestinationArn: !GetAtt LogSubscriptionProcessor.Arn
          FilterPattern: "?error Error"
          LogGroupName: /aws/lambda/my-lambda
        DependsOn: AlertProessPermission   

### Policy to send e-mail

    EmailPolicy:
        Type: AWS::IAM::ManagedPolicy
        Properties:
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - ses:SendEmail
                Resource: "*"

This policy you need to assign to the role under which lambda function is running.

This is all you need. In this set up , we have subscribed to cloud watch log group of lambda - *my-lambda*. So whenever this lambda writes any event which has key word '**error**' of '**Error**', cloud watch will invoke lambda function *Log_Subscription_Processor. *This lambda, after reading the event data, sends an email using amazon simple email service and you get the notification.

In next article, we will see how we can use slack to send those notifications.

ref:
[

Using CloudWatch Logs Subscription Filters - Amazon CloudWatch Logs

Associate a subscription filter with a log group containing AWS CloudTrail events.

![](https://docs.aws.amazon.com/assets/images/favicon.ico)Amazon CloudWatch Logs

](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/SubscriptionFilters.html)[

Filter and Pattern Syntax - Amazon CloudWatch Logs

Match terms in log events using metric filters in CloudWatch Logs.

![](https://docs.aws.amazon.com/assets/images/favicon.ico)Amazon CloudWatch Logs

](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html)
