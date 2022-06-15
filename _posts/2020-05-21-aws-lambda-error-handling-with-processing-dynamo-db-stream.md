---
title: AWS lambda error handling with  DynamoDB stream processing
slug: aws-lambda-error-handling-with-processing-dynamo-db-stream
date_published: 2020-05-21T09:32:28.000Z
date_updated: 2020-05-21T09:32:28.000Z
---

If you are using DynamoDB in your application, chances are that you are also utilizing [DynamoDB  stream](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html) to process any changes in the records. Here I'll just explain how you can handle the error while processing the DynamoDB stream with minimum/no code. I'm using cloud formation template to deploy all the changes to aws stack

if you are already processing stream, you must have created an EventSourceMapping for lambda and stream. If not,  here is an example of how it looks like

    MyDynamoTableStreamMap:
        Type: AWS::Lambda::EventSourceMapping
        Properties:
          BatchSize: 100
          Enabled: True
          EventSourceArn: !GetAtt MyDynamoTable.StreamArn
          FunctionName: !GetAtt MyStreamProcessor.Arn
          StartingPosition: LATEST 
    

EventSourceMaping
Now , we want to send all the failures to an SQS. For that we need to create an SQS and update our EventSourceMapping to include that.

    FailedRecordsQueue:
        Type: AWS::SQS::Queue
        Properties:
          QueueName: My_Failed_records_Queue

Create SQS
    MyDynamoTableStreamMap:
        Type: AWS::Lambda::EventSourceMapping
        Properties:
          BatchSize: 100
          Enabled: True
          EventSourceArn: !GetAtt MyDynamoTable.StreamArn
          FunctionName: !GetAtt MyStreamProcessor.Arn
          StartingPosition: LATEST
          DestinationConfig: 
            OnFailure: 
              Destination: !GetAtt FailedRecordsQueue.Arn
          MaximumRetryAttempts: 0

EventSourcemapping includes SQS as Destination
You also need to assign the SendMessage permission to the role under which lambda is running. You can assign this policy to the role .

     SQSPolicy:
        Type: AWS::IAM::ManagedPolicy
        Properties:
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - sqs:SendMessage*
                Resource:
                  - !GetAtt FailedRecordsQueue.Arn

The DestinationConfig provides the SQS as destination for failure. Here I have kept MaximumRetryAttempts as 0 so that lambda does not retry and send it to the SQS immediately. That's all you need to do to achieve this.

While testing this, if you have been using lambda, you may create a test event from the aws console to trigger lambda causing error and see if it writes to queue. That will not work. Since this is specially handling failure while processing DynamoDB stream, you need to change data in DynamoDB table to generate a stream which causes a failure within lambda and then it writes to SQS. You can go to created queue and check,  it will have not the record which caused failure but enough information to fetch data from shards.

Ref : [https://aws.amazon.com/about-aws/whats-new/2019/11/aws-lambda-supports-failure-handling-features-for-kinesis-and-dynamodb-event-sources/](https://aws.amazon.com/about-aws/whats-new/2019/11/aws-lambda-supports-failure-handling-features-for-kinesis-and-dynamodb-event-sources/)
