---
title: How not to use Athena !!!
slug: how-not-to-use-athena
date_published: 2020-11-12T14:19:06.000Z
date_updated: 2020-11-12T14:19:06.000Z
tags: aws, athena, bigdata
---

For those who don't know, AWS Athena is a query service that makes it easy to read data stored in S3 bucket using SQL queries. It is optimized for querying huge amount of data and you don't even need to set up any infrastructure. But little did I know, it is not a replacement for database !!!

At work, we process huge amount of data and store that in S3. We use Athena to access that for various applications. Most of the use case require reading from a single table /few joins which works well. 

    SELECT * FROM MyTable;

But queries which involves join across multiple tables or views, using function like group by, order by etc, Athena couldn't perform. Query planning time for these queries was way too high and actual execution time was only a fraction of it. We tried all the [query optimization tips](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/) which AWS recommends which it didn't help.

For example, a query which takes about a seconds in SQL, took more than 15 seconds in Athena. You can easily find the breakup of the time a query takes using [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/athena.html). When you execute a query in athena, you get a query execution id. Use that id to get statistics.

    client = boto3.client('athena',region_name=region)
    client.get_query_execution(
        QueryExecutionId='12345-aabb-4512-56788-12345'
    )

Response of this contains statistics for query with details about where time was spent.

    'Statistics': {'EngineExecutionTimeInMillis': 17912,
       'DataScannedInBytes': 3307921,
       'TotalExecutionTimeInMillis': 11152,
       'QueryQueueTimeInMillis': 210,
       'QueryPlanningTimeInMillis': 15100,
       'ServiceProcessingTimeInMillis': 30},
      'WorkGroup': 'primary'},

Every time a query is fired, at a high level following steps are performed.

1. Athena puts the query in a queue. If this is taking too much time in queue, you have the option of increasing the no. of parallel queries.
2. Query planning . Athena makes call to glue to get the metadata about tables including partitions, s3 locations etc.
3. Execute the plan on distributed nodes.
4. Combine the result and write to S3.

#2 was where we thought athena is spending time . I'm not sure whether it is number of calls or the way these are performed, it takes a lot of time to complete that. It was linearly increasing with number of tables in the query. The metadata about these tables and view does not change that frequently, if this can be cached, queries will run much faster but since athena is stateless, it can not cache the metadata. 

The queries that we were writing has multiple layers of views and tables which was the problem. Since SQL could perform so we never had any issue but with athena, **we had to change the way we write queries. **

1. **Get rid of multiple layer**. View using another view using table. Instead, if possible, just use table.
2. **Pre compute** If the query performs a lot of processing, consider materialize that and use computed data in query.

With these 2, we were able to bring back the execution time under 2 sec for the queries which were taking ~15 seconds.

Learning:

1. Athena is not a low latency service. It is not suitable for backend API if you need highly responsive system.
2. Keep the queries simple. Know that for each table/view, it will get the metadata from glue and that time will increase with number of tables and views. 
