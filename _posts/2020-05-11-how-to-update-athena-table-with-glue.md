---
title: How to update athena table with glue
slug: how-to-update-athena-table-with-glue
date_published: 2020-05-11T16:39:21.000Z
date_updated: 2020-05-11T17:06:19.000Z
tags: aws, athena, python, glue
---

I was working on an application where data was stored in s3 bucket and athena was used to query this data. Since data was frequently updated (on an average, every 30 min or so), which wipes the data in s3 folder and writes new set of data,  I had to make sure that during updates, athena queries should not return inconsistent results.

So now, we are maintaining two locations in s3 with contains the same data. Let's call these location L1 and L2. When we update data at L1, we make all the athena tables  point to L2 and vise versa.Pointing athena table to a location is a trivial task. A simple query does this.

ALTER TABLE <table name> SET LOCATION <new location>

This query, though simple , may take a lot of time to complete.  Depending on the limit of parallel queries, this query may go in a queue and take more time to complete.

Another options is to use glue api to update the table. Glue client contains a method **update_table**.  Unlike other methods to update, this method required minimum fields (other than field to be updated) to be supplied in the request in order to update the table. If you just pass the location field with a new value to this method, athena throws all kinds of exceptions starting from null pointer exception.

1. Get the metadata of the table

    import boto3
    client = boto3.client('glue')
    response = client.get_table(
        CatalogId='string',
        DatabaseName='string',
        Name='string'
    )
    

2. Update the table using [update_table](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html#Glue.Client.update_table) api

    response_update = client.update_table(
        DatabaseName=<db name>,
        TableInput={
            'Name':<table name>,        
            'StorageDescriptor':{
                'Columns':response['Table']['StorageDescriptor']['Columns'],
                'Location':<s3 location>,
                'SerdeInfo':response['Table']['StorageDescriptor']['SerdeInfo'],
                'InputFormat':'org.apache.hadoop.mapred.TextInputFormat',
                'OutputFormat':'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'           
            },
            'PartitionKeys':response['Table']['PartitionKeys'],
            'TableType':'EXTERNAL_TABLE',
            'Parameters': {'EXTERNAL': 'TRUE'}
        })

In the above code block, since I have to update only table location, I'm passing all the values except location from get_table response. These are the minimum fields required for the update api.

This query comparatively takes less time and I was able to complete the whole operation in ~1 sec. That worked for me.

for further reading , refer to glue [api](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html) documentation
