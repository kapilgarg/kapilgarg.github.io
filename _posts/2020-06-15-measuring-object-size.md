---
title: Measuring object size with getsizeof
slug: measuring-object-size
date_published: 2020-06-15T04:09:53.000Z
date_updated: 2020-06-15T04:09:53.000Z
tags: python, getsizeof, programmimg
---

One of my colleague was running a batch to insert records in AWS firehose stream and getting service unavailable exception continuously. Probably, batch was inserting more data than allowed per second. So We thought of adding a log statement which prints the size of records which are sent to firehose.

After adding the log statement, we restarted the process and when we checked the logs for object size, all the objects were of same size !!! **2464 bytes**. We were using [put_record_batch](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/firehose.html#Firehose.Client.put_record_batch) API and we fixed the batch size to 300 but all the objects were not of the same size in a given batch. This is certainly not correct because the size should be somewhere close to 1 MB.

This was happening because of how [getsizeof](https://docs.python.org/3/library/sys.html#sys.getsizeof) works. 

This was the log statement added to print the size of records in bytes

    logger.info('batch size = ' + str(getsizeof(batch)))
    
    firehoseClient.put_record_batch(DeliveryStreamName='my_delivery_stream', Records=batch)

here, variable *batch* is a list which contains the objects to be sent to firehose. 

The way [getsizeof](https://docs.python.org/3/library/sys.html#sys.getsizeof) works is - it gets you the size of objects it **contains **and **not the one it refers to**. In this case, since list contained 300 objects, it was returning size of 300, 8 byte pointers added with the default size of list.

    #size of an empty list
    >>> from sys import getsizeof
    >>> getsizeof([])
    64
    
    # a temporary class for demo
    >>> class Temp():
    ...     pass
    
    #size of list containing a single object 
    >>> getsizeof([Temp()])
    72

Size of an empty list is 64 bytes and when an object is added to it, size of that list only increased by 8 bytes. because it contains only the pointer to this object and not the actual object.

To get the size including the all the objects which this list refers to, you can use this recipe which gets the size of each object in the list and add that to the size of list.
[

Compute Memory footprint of an object and its contents « Python recipes « ActiveState Code

![](https://code.activestate.com/static/activestyle/img/favicon.ico)Jean Brouwers

![](https://code.activestate.com/static/img/bullet_disk.png)
](https://code.activestate.com/recipes/577504/)
If you are interested, some more reading on the same 
[

Understand How Much Memory Your Python Objects Use

Python is a fantastic programming language. It is also known for being pretty slow, due mostly to its enormous flexibility and dynamic features. For many applications and domains it is not a...

![](https://static.tutsplus.com/assets/apple-touch-icon-edce48c2c1293474a34fa8f1c87b5685.png)Gigi SayfanEnvato Tuts

![](https://cms-assets.tutsplus.com/uploads/users/1199/posts/25609/preview_image/python.png)
](https://code.tutsplus.com/tutorials/understand-how-much-memory-your-python-objects-use--cms-25609#:~:text=An%20empty%20list%20takes%2072,an%20int%20is%2024%20bytes.)
