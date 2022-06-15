---
title: Pyspark on AWS Fargate
slug: running-pyspark-on-aws-fargate
date_published: 2021-01-17T11:30:50.000Z
date_updated: 2021-01-17T11:30:50.000Z
tags: pyspark, aws-sdk, hadoop, fargate
---

I'm using AWS Batch to run a few pyspark batch jobs and many times, when a job is submitted, it takes a few minutes to start the job. This delay may range from 2-15 minutes depending on the availability of EC2 machine and on the configuration provided. This is something I'd definitely want to avoid. Recently AWS also started supporting Fargate as computing resource which means if I use Fargate, I can simply specify my requirements (cpu, memory ) and don't have to worry about queuing , priority, retries etc.

Fargate requires a bit of different [configuration](https://docs.aws.amazon.com/batch/latest/userguide/fargate.html) than AWS batch. We need to provide some additional configuration on compute resource and job definition and some existing configuration for same is not applicable for Fargate. 

If you are running spark 2.4, your existing spark code will throw exception when accessing any of the AWS API like reading from S3/ writing to S3 etc.
![](/content/images/2021/01/image.png)
The reason being, spark 2.4 uses AWS SDK 1.7.4 which can not read credentials for Fargate. This SDK version was released before fargate was supported hence it is not able to load the credentials from the IM role that Fargate uses.

Many versions of this SDK have been released which now supports Fargate but we can not simply use an older version of spark with newer version of AWS SDK. Each version of Hadoop is built with specific version of SDK. In this case, we will have to upgrade spark-hadoop so that it uses a newer version of AWS SDK.

The latest release of spark is 3.0.1 and  there is a pre built package with Hadoop 3.2. We can use this package to upgrade the spark version and use an updated AWS SDK which is compatible with this spark-hadoop package.

With this informatin at hand, we can build a docker image and use that with Fargate. While doing so, do not install pyspark using pip. It deploys older version of spark and hadoop which will throw all kinds of errors with newer version of AWS SDK.  Since pyspark is already available with this spark-hadoop package, you just need to provide the location of that to the python and every thing will fall in place.

1. There are two ways to do that. If you are not aware of the location,use a python package *findspark*. import this as a first step in your process and call init on this . This will set up the SPARK_HOME environment variable and python will be able to import pyspark.
2. If you are already aware of the path for pyspark, you can directly set that when creating docker image. 

Thats all required to run your spark job on Fargate.

 ref : 

1. [Downloads | Apache Spark](https://spark.apache.org/downloads.html)
2. [Apache Hadoop 3.2.0 – Dependencies Report](https://hadoop.apache.org/docs/r3.2.0/hadoop-project-dist/hadoop-hdfs/dependency-analysis.html)
3. [https://aws.amazon.com/blogs/aws/new-fully-serverless-batch-computing-with-aws-batch-support-for-aws-fargate/](https://aws.amazon.com/blogs/aws/new-fully-serverless-batch-computing-with-aws-batch-support-for-aws-fargate/)
