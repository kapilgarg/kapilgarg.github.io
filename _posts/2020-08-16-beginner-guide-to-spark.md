layout: post  
title: "beginner-guide-to-spark"  
date: 2020-08-16  
categories: Python apache-spark  

# Your first spark application

Apache spark is a framework with which you can process huge amount of data with lightening fast speed. You can run it on a single node or in a cluster where task is distributed among nodes. One of the usage of spark is in ETL process where you extract data from a source, transform as per your needs and then load it to destination for consumption. Here I'd like to explain how to write a basic spark application

Use case 
Suppose you are running an application on AWS which is generating log files in S3 bucket. You also have an analytics application which can analyze these log files and produce some meaningful information. These files in their raw form are not something that analytics application can consume. So We extract this data from S3 Bucket , transform it the way it can be consumed and then load to another location in S3.

Spark provides native bindings for the Python, Java,  Scala and R. Here I'm using [pyspark](https://spark.apache.org/docs/latest/api/python/index.html) for this article.

## Before we could start
There are some terms we need to be aware of.

### Spark session

First step in any spark application is to create a spark session. SparkSession provides a single point of entry to interact with underlying Spark functionality and allows interacting with DataFrame and Dataset APIs.

```
From pyspark.sql import SparkSession

spark = SparkSession
.builder
.appName(‘my_spark_application’)
.getOrCreate()
```
### RDD(Resilient Distributed Datasets)
Another concept you may hear a lot is RDD. RDD is a fault tolerant collection of elements which can be processed in parallel. Fault tolerant here refers to the idea that in case of failure, RDD can be recreated by spark.

This is enough to get started. 

## Extract
To extract the data, we will need to read files from the source location.
```
file_path = "s3a://my-s3-bucket/data/2020/08/14/app-log.gz"

spark_context = spark.sparkContext
log_rdd = spark_context.textFile(file_path)
```

This input file contains data in json format with gzip compression that's why I've called .textfile(<args>). If you are reading parquet files, you will need to call appropriate method to read that.

Spark supports multiple compression type and you don't need to provide any additional information when reading a file. It is taken care by spark.

## Transform
Now that we have log_rdd, we can do transform that are required. Suppose we need to generate a new field based on some other fields.

Lets define out transform method.
```
def transform_rdd(rdd):
	def generate_field(record):
    		record['new_field'] = generate_new_field()
        	return record
    return rdd.map(lambda r : generate_field(r))
 
 log_rdd = transform_rdd(log_rdd)
 ```
 This transform_rdd function here is adding a new field in the existing record. I have kept it very basic for the sake of understanding but you can do any transformation required on the records here. RDD provides a map function which takes another function as input and apply that on each record in the RDD. The result of map is also an RDD.

 ## Load
 When we are done with the transformation, we need to load this data to destination. When writing the data to destination, you can partition the data. Partitioning means arranging your data by the values of a particular field. For example, all the log records may contains a field called createdOn which is essentially the date when the log was written. If you choose to partition on createdOn, spark will segregate log in separate files (by createdOn) and these files will be written to separate directories(with name as createdOn=<date>)

Before writing , we need to convert the RDD to a data frame. we had used RDD because it is a low level datasets which provides may operation not available on data frame.

To create data frame, we can either pass the schema of the record or create it without schema. If we don't provide the schema, spark infers the schema that may not always be 100% accurate . So if possible, provide the schema. Just to give you an example of schema, if you have a json with 3 fields, schema will be of following shape.
```
{
	'url': <string>,
    	'is_final':<boolean>
    	'score':<int>
}

#schema 
from pyspark.sql.types import StructType, StringType, IntegerType, StructField, BooleanType
schema = StructType([
        StructField("url", StringType(), True),
        StructField("is_final", BooleanType(), True),
        StructField("score", IntegerType(), True))
```
```
# with schema
log_df = spark.createDataFrame(log_rdd, schema))

# without schema 
log_df = spark.createDataFrame(log_rdd))
```
## Save data frame with partition
```
log_df.write.partitionBy('createdOn).json(path, compression=None)
```

When you are writing the data frame, the path that you provide should be a directory path and spark will generate the folders as per partition and file name when writing.

This is a very basic code that I have put together. Spark provides a lot of features to customize the behavior and there are a lot of setting that you can use to configure the behavior.

you can experiments with different kinds of file formats and compression, you can control the size of each file and also partition on multiple fields. All of this depends on how are you going to use the final data set.

Next we will see how to run this spark process on AWS.

Till then ... happy coding .
