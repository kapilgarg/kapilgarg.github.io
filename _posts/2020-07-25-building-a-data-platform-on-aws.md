---
title: Building data platform on AWS - part 1
slug: building-a-data-platform-on-aws
date_published: 2020-07-25T10:44:56.000Z
date_updated: 2020-07-25T10:45:27.000Z
tags: aws, platform, design
---

You have joined this startup which wants to build a brand new,  scalable platform. With this it aims to reduces its operating cost and provides  better services to its customer. 

The platform in question here is the entire back end system to support  their products.  Since you don't want to manage all the infrastructure,  the obvious choice is cloud. So you call your team, cloud gurus for discussion. After some brain storming sessions, team come to conclusion that AWS is the platform which you can use. That wasn't so difficult after all. AWS is scalable, cost effective  and  you are already familiar with it!
![](/content/images/2020/07/image-12.png)
---

## Data

The data, this company deals with, is dynamic in nature.  With SQL/Relational DB, we are  going to have tough time. Luckily AWS provides d*ynamodb* - a No SQL, highly scalable database. That would be a good choice to store the data. 

Products/services use the data in two ways -

1. **Low volume / real time read** - As soon as data is updated, it should be available for consumption. dynamodb can do that. 
2. **High volume / near real time read - **Reading huge quantity of data but it requires *near *real  time data. That means , if data is updated now and it is available  after some time (lets say 30 mins from now), that is OK.

Now for 2nd use case, dynamodb  will be overkill.  You may ask why!!!

Querying huge quantity of data from dynamodb will be costlier. Since real time access is not a concern, we can think of something cost effective (and  serves the purpose).

It turns out that AWS also provides S3 which  is way cheaper than dynamo and can also store huge quantity of data. It also makes  data available for consumption by means of Athena which  is another offering on AWS.

So we decide to keep the data in two  places. In dynamodb for low volume / real time read and in S3 for high  volume / near real time (and we save some money also).
![](/content/images/2020/07/image-9.png)
---

## Sync

But now we have a problem. How do we move data from dynamodb to S3 in near real time? As soon as the data comes to dynamodb, it should also be made available in  S3 which might take some time. After all its a near real time.

Any  service which is writing data to dynamo db can also write to S3 bucket.  What if writing to dynamo fails?  Would it still write to S3? or if  Write to dynamo succeed but to S3 fails?  Should this be a synchronous  operation . In that case writing operation will end up taking more time.  If we keep this asynchronous, we will have to track all the failures...  a lot of cases to consider!!!

AWS did the magic again. dynamodb  provides 'real time updates in records' through dynamodb stream . It is more like a pub-sub mechanism where dynamodb publish all the changes.  Client can  subscribe to it and take action (invoke lambda). So we configure this stream  on table and attach a lambda function to it. When ever any record is  inserted/updated, that record is available in stream.  It also triggers the lambda function with those records as arguments to this lambda.

So  now we have a table which produces a data stream whenever any record is  changed in that . That data stream triggers the lambda which writes  these records in s3. So far so good.
![](/content/images/2020/07/image-10.png)
---

Writing data with lambda function to S3 will be inefficient here. because you will be making frequent trips with very small amount of data. It would be better if we could write the data in batches.

We create a firehose stream to write dynamodb stream data to S3. firehose buffers the records for a given duration before writing those to destination. It is also integrated with S3. So that you can specify an S3 location as destination when you create this stream and forget about that. you just keep pushing the data to firehose stream and it will deliver the data to destination.
![](/content/images/2020/07/image-11.png)
---

If you haven't worked with S3, it is an object store where objects are immutable. What that means is , in s3, we will be writing records in file. and any operation is performed in that file and not at the individual record.

For example, If we push 10 records to firehose,  firehose writes all those 10 records to s3 as an object. Like any database , we can not  manipulate the individual records. It is the object that we can read / write.

That introduces some complexity. dynamodb contains only one copy/record which is the latest one. But each insert/update is an individual record in dynamodb stream. If we update a record twice, dynamodb stream will contain two individual copy of that record. Each copy will belong to different state. Hence we need to make sure that we update the correct object in S3 to keep only latest copy of record. Since S3 objects are immutable, we need to read the object, update the record and then re write the object back to S3.

To handle this, we introduce Apache Spark . Spark is capable of handling large quantity of data efficiently. We create a spark job to process  input files/data written by firehose and run it on aws batch. Spark Process updates any existing record with latest record or add a new record to S3 object. 

With this setup , our data is now available in S3 with only latest copy and no duplicates.  Now the only part remaining is how to query this?

Here athena comes for our rescue. In athena, we can create SQL like table which points to data in S3. We can query these table with SQL like query. Please note that these tables provide read only access to data. You can not update data using these tables. These are good for querying and believe me they perform better with more data . But if you have to update any record, that update has to come from the dynamo and has to pass through the whole chain. This is the tread off for keeping the cost low.
![](/content/images/2020/07/1-1.png)
This completes the initial design of this platform. In next posts, I'd like to go a little deeper in spark job and list down some of the issue we face with this and the solutions.
