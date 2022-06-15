---
title: How to detect drift in AWS stack- part 1
slug: how-to-detect-drift-in-aws-stack
date_published: 2020-06-28T16:09:02.000Z
date_updated: 2020-06-28T16:11:41.000Z
tags: aws, cloudformation, drift
---

If you have deployed AWS stack using cloud formation template, there is a possibility that someone may have changed some of the properties on one or more resources without updating the CF template. As a result of this, your actual deployed resource is different from what is defined in the template . This is called drift.

For example, you may have a lambda function which takes a parameter , say log-level and you have declared that in your template as 1. Some one with enough permission on that stack wanted to do a quick test by changing the value to 2 and didn't revert to original value. Now your AWS stack is not the same as template which was deployed. Cloud formation would still think that it has the old value (since template hasn't changed) but the actual value is different.

### But how does that affect you? 

Most of the time, you will be looking at the template to see what have you deployed in the AWS environment and not the actual and this may give you wrong information. Off course, you can check the  drift from AWS console. 

First I'll show you how you can check drift manually. Later we will automate this process.

Lets create a stack with single resource for simplicity , introduce a drift and see how to check that. In this example, I'm creating an S3 bucket with versioning Enabled and later manually disabling that to introduce the drift.

From the cloud formation console, you can deploy my-stack.yaml for this example.

    --- 
    
    Resources: 
    
      MyS3Bucket: 
    
        Properties: 
    
          BucketName: my-fav-collection
    
          VersioningConfiguration: 
    
            Status: Enabled
    
        Type: "AWS::S3::Bucket"

### To check the drift manually -

Go to [https://<region>.console.aws.amazon.com/cloudformation/home](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1)
![](/content/images/2020/06/image.png)
Click on the stack you created 
![](/content/images/2020/06/image-1.png)
Expand *Stack actions* drop down and choose *Detect Drift*. This will initiate drift detection and might take some time.
![](/content/images/2020/06/image-2.png)
From the same menu, select *View drift* result and it will show you the resources in AWS stack and their status .

If there is no change, It will show as IN_SYNC else MODIFIED.
![](/content/images/2020/06/image-3.png)
To view the details for Any drifted resource, click that radio button and click on View Drift Details button. It will show you the properties, their expected and actual value.
![](/content/images/2020/06/image-4.png)![](/content/images/2020/06/image-5.png)
That's it. 

In next post, I'll show you how you can set this whole process so that you don't have to do this manually.
