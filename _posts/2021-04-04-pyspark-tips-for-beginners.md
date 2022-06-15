---
title: PySpark tips for beginners
slug: pyspark-tips-for-beginners
date_published: 2021-04-04T10:13:57.000Z
date_updated: 2021-04-04T10:13:57.000Z
tags: python, pyspark, programming, apache spark
---

### Be careful when you use .collect()

Do not call *.collect()* on RDD or data frame. Your driver may go out of memory if RDD or data frame is too large to fit on a node. Use *take() *function instead. You can specify the count with *take* that reduces the number of records returned.

    rdd.collect() 	# do not use this unless you are sure.
    rdd.take(10)	# use this. It samples the data

---

### Check if rdd or data frame is empty

To check if rdd is empty, use *isEmpty(). *To check if dataframe is empty, use len(df.take(1))

Do not use .count() to check if rdd/df contains any records or not. Use count only when you need to get the exact number of records.

    #do this 
    rdd.isEmpty()			# check for empty rdd
    if len(df.take(1)):		#check for empty dataframe
    	do_something()
        
    #instead of this 
    rdd.count()

---

### To overwrite a partition

When writing the data to destination, if you want to overwite the existing partition's data, set "*spark.sql.sources.partitionOverwriteMode*" to  "*dynamic*" in spark session config. if you don't do this, spark may complain that *path already exists*.

    spark = SparkSession.builder
            .appName("my_app")\        
            .config("spark.sql.sources.partitionOverwriteMode", "dynamic")\       
            .getOrCreate()

---

### reduceByKey or groupByKey

When you need to agreegate data based on key, avoid grouopByKey since it result in shuffle of data from multiple partitions and may cause out of memory error. Use reduceByKey as it combines the data for a key hence amount of data shuffle is less.

     >>> words = ["one", "two", "two", "three", "three", "three"]
     >>> word_rdd = sc.parallelize(words)
     >>> word_rdd = word_pair_rdd.map(lambda word : (word,1))
     
     # groupByKey 
    >>> word_rdd.groupByKey().map(lambda a:(a[0],sum(a[1]))).collect()
    [('two', 2), ('three', 3), ('one', 1)]
    
    # reducyByKey
    >>> word_rdd.reduceByKey(lambda a,b:a+b).map(lambda x:(x[0][0],x[1])).collect()
    [('two', 2), ('one', 1), ('three', 3)]

---

### Cache rdd

spark is lazy evaluation. Every time you call a function which needs data, it computes the rdd and that is expensive operation. Instead, cache/persist rdd so that it is computed only once and subsequent operation will not require spark to recompute rdd. caching an rdd will make a huge impact in the performance but make sure that you are using the same rdd across multiple operation and there is no transformation from one operation to other.

    rdd.cache()
    rdd.count() # at this point, rdd will be cached hence this will take regular time
    rdd.collect() # this will use cached rdd, this will be many times quick

You can either use cache() or persist. cache only stores in memory while with persist, you can specify where to store (disk/memory)

---

### Reduce partitions in a cluster

By default, spark creates as many partitions in a cluster as the number of cores. You may need to update these partition to optimize the resources. If you have less partitions than requird, you will not be using your resources fully. If you have more number of partitions, there will be overhead of shuffling the data. 

In case you have to reduce the partitions, prefer *coalesce *over repartition. coalesce minimize the data shuffle hance a better choice.

---

This is not an ultimate guide to Spark optimization. There are plenty of other such tips which we haven't covered here. We will continue doing that in a separate post.
