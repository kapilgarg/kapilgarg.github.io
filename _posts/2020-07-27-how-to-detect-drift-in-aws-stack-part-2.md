---
title: How to detect drift in AWS stack- part 2
slug: how-to-detect-drift-in-aws-stack-part-2
date_published: 2020-07-27T16:22:40.000Z
date_updated: 2020-07-27T16:22:40.000Z
tags: aws, cloudformation, stack, drift, lambda, python
---

In the [first part](/how-to-detect-drift-in-aws-stack/), I had explained how to detect drift manually. Now we will see how to automate that.

We are going to write a lambda function which can tell us whether there is any drift present in a given stack. There are boto3 api which helps to get this information. We can schedule this lambda to run at a given schedule or can run it on demand to check for drift.

### Drift

    import boto3
    import pprint
    
    client = boto3.client('cloudformation')
    pp = pprint.PrettyPrinter(indent=4)
    
    response = client.detect_stack_drift(
        StackName='my-test-stack'    
    )
    pp.pprint(response)

response 

    {   'ResponseMetadata': {   'HTTPHeaders': {   'content-length': '365',
                                                   'content-type': 'text/xml',
                                                   'date': 'Mon, 27 Jul 2020 '
                                                           '14:20:45 GMT',
                                                   'x-amzn-requestid': '9816363a-f195-4e16-981f-e9d58a943a05'},
                                'HTTPStatusCode': 200,
                                'RequestId': '9816363a-f195-4e16-981f-e9d58a943a05',
                                'RetryAttempts': 0},
        'StackDriftDetectionId': '5c644560-d014-11ea-8be2-027feff0383e'}

Response gives a 'StackDriftDetectionId' which we are going to use to get more information.

### Drift status

    drift_detection_response = client.describe_stack_drift_detection_status(
        StackDriftDetectionId='5c644560-d014-11ea-8be2-027feff0383e'
    )
    
    pp.pprint(drift_detection_response)

and the response -

    {'StackId': 'arn:aws:cloudformation:ap-south-1:123456789:stack/my-test-stack/d7d22f50-b70f-11ea-b1ed-0207d27cfd0c',
     'StackDriftDetectionId': '5c644560-d014-11ea-8be2-027feff0383e',
     'StackDriftStatus': 'DRIFTED',
     'DetectionStatus': 'DETECTION_COMPLETE',
     'DriftedStackResourceCount': 1,
     'Timestamp': datetime.datetime(2020, 7, 27, 14, 20, 45, 622000, tzinfo=tzutc()),
     'ResponseMetadata': {'RequestId': '13b04fef-7628-49f9-a4d6-d0fb65651efd',
      'HTTPStatusCode': 200,
      'HTTPHeaders': {'x-amzn-requestid': '13b04fef-7628-49f9-a4d6-d0fb65651efd',
       'content-type': 'text/xml',
       'content-length': '780',
       'date': 'Mon, 27 Jul 2020 14:24:03 GMT'},
      'RetryAttempts': 0}}

StackDriftStatus of this stack is "DRIFTED".  

### Drift details

Now to get the details about the drift, we can query the stack for all the resource where drift status is modified.

    response = client.describe_stack_resource_drifts(
        StackName='my-test-stack',
        StackResourceDriftStatusFilters=[
            'MODIFIED',
        ]
    )

This response gives us a list of all the drifts in the stack. Each object in the list, among other properties, gives you ExpectedProperties, ActualProperties and PropertyDifferences.

    { 
    	'ActualProperties': '{"BucketName":"my-fav-collection","VersioningConfiguration":{"Status":"Suspended"}}',
        'ExpectedProperties': '{"BucketName":"my-fav-collection","VersioningConfiguration":{"Status":"Enabled"}}',
        'LogicalResourceId': 'MyS3Bucket',
        'PhysicalResourceId': 'my-fav-collection',
        'PropertyDifferences': [   {   'ActualValue': 'Suspended',
                                       'DifferenceType': 'NOT_EQUAL',
                                       'ExpectedValue': 'Enabled',
                                       'PropertyPath': '/VersioningConfiguration/Status'}],
        'ResourceType': 'AWS::S3::Bucket',
        'StackId': 'arn:aws:cloudformation:ap-south-1:844280065683:stack/my-test-stack/d7d22f50-b70f-11ea-b1ed-0207d27cfd0c',
        'StackResourceDriftStatus': 'MODIFIED',
        'Timestamp': datetime.datetime(2020, 7, 27, 14, 20, 46, 478000, tzinfo=tzutc())
        }

Here, PropertyDifferences gives the propertyPath, its Actual value and ExpectedValue. With this we can identify where the stack is not in sync with cloud formation template.

### Complete code

    import boto3
    client = boto3.client('cloudformation')
    
    def drift_status(stack_name):
            response = client.detect_stack_drift(StackName=stack_name)
            drift_detection_status = client.describe_stack_drift_detection_status(
                                            StackDriftDetectionId=response['StackDriftDetectionId']
                                        )
            if drift_detection_status['StackDriftStatus'] == 'DRIFTED':
                stack_resource_drifts_response = client.describe_stack_resource_drifts(
                            StackName=stack_name,
                            StackResourceDriftStatusFilters=['MODIFIED',])
                return stack_resource_drifts_response['StackResourceDrifts']
            return None

Next, we will schedule this lambda to run regularly and update you about any drift.
