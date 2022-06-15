---
title: Spark Streaming for beginners
slug: spark-streaming-for-beginners
date_published: 2021-06-13T18:34:29.000Z
date_updated: 2021-06-13T18:53:37.000Z
tags: pyspark, streaming, python, apache spark
---

Whether you are running an eCommerce store and want  to put up a dash board which shows the number of  orders processed every minute or run a very popular blog and would like to display trending articles on your web site or any other scenarios like this, all of these require the processing of data in real time. We may be getting thousands of orders per minutes or hundreds of thousands of page view per minutes. What we need to do is process that stream of data on the fly so that the result is available with minimum latency.

**Spark streaming** is capable of processing data stream in a *distributed manner with high throughput and scalability*.  It can read the stream from multiple sources like files, over TCP connection, Kafka, Kinesis etc. Lets take an example of eCommerce store and see how to get the numbers of orders processed per minute from continuous stream.

We have a TCP server which acts as source for stream. Spark Streaming app connects to this server to receive the data. In real world, this TCP server may be replaced with Kafka, Kinesis etc. 

Once we start the TCP server and start spark app, spark app starts reading stream from server. Here is the spark code and records look like when spark reads it.

    session = SparkSession.builder.master("local[2]")\
            .appName("PythonStreamingOrderCount")\
            .config("spark.ui.showConsoleProgress", "false")\
            .getOrCreate()
    
    sc = session.sparkContext
    sc.setLogLevel("ERROR")
    ssc = StreamingContext(sc, 5)
    
    orders = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
    orders.pprint()

    # start the TCP server
    python server.py
    
    #start spark app
    python .\order_count.py localhost 9998

![](/content/images/2021/06/image.png)
ssc is the streaming context which reads the stream from source every 5 seconds. Object *orders *is of type *pyspark.streaming.dstream.DStream. *This is an abstraction provided in pyspark streaming api and is represented as sequence of Rdds. Each record in the DStream is an Rdd. 

This is only input stream without any transformation. If we want to get the numbers of orders by their status every 5 seconds, we can apply some higher level functions on this DStream and process this data.

    session = SparkSession.builder.master("local[2]")\
            .appName("PythonStreamingOrderCount")\
            .config("spark.ui.showConsoleProgress", "false")\
            .getOrCreate()
            
    sc = session.sparkContext
    sc.setLogLevel("ERROR")
    ssc = StreamingContext(sc, 5)
        
    orders = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
    
    orders = orders.map(lambda rdd: json.loads(rdd))\
    .map(lambda r: (r['status'], 1))\
    .reduceByKey(lambda a, b: a+b)
    
    orders.pprint()
    
    ssc.start()
    ssc.awaitTermination()

Since each rdd is a string, we convert that to a dictionary, group by order status and add the frequency of each order in that group.here is the output.
![](/content/images/2021/06/test.gif)
Every 5 second window, spark process the receive records and calculates the how many orders were sent by their status.

You could also get the updated total count of all the orders by status in addition to the current 5 minutes count.

    def update(new_values, current_values):
        current_values = current_values or 0    
        return sum(new_values, current_values)
     
     total_count = orders.updateStateByKey(update)
     total_count.pprint()

update function keeps the state updated by adding new values to the current values so that we have a cumulative count available for each key (status)
![](/content/images/2021/06/image-2.png)
### High level components of spark streaming 
![](/content/images/2021/06/spark-streaming-1.png)
We will look at other API of spark streaming and how those can be used in different use cases next time. 

source code for this sample is available at [https://github.com/kapilgarg/spark-streaming-samples](https://github.com/kapilgarg/spark-streaming-samples)

Ref: [https://spark.apache.org/docs/latest/streaming-programming-guide.html](https://spark.apache.org/docs/latest/streaming-programming-guide.html)
