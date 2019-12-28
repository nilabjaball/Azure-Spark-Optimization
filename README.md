
# Spark Optimization -- Few Quick Wins

Spark platform is a  distributed  in-memory cluster computing system that speed up  the  processing  of big-data analytic applications. A Spark job can load and cache data into memory and query it repeatedly.

Some of the common challenges of  long running Spark jobs or need of large capacity of Cluster size are  
memory pressure, because of improper configurations, long-running operations, and  concurrent workloads 

There focus of this article is how the data engineers or developers can follow some common practices to speed up the spark jobs or build a cost effective solution.   The infrastructure pieces such as container\executor size , compute nodes etc are not covered.  

For that you can use 

For Data bricks sizing ,  you can use the  best practices guide
https://github.com/Azure/AzureDatabricksBestPractices/blob/master/toc.md

For HDI,  refer the article https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-performance-tuning-spark

For Data Management and Coding guidelines, here are some of the useful tips and tricks 


## Data Partitioning 

### Story of partitioning and re-partitioning  

Distributing data to increase the  parallelism  is one of the important concept that engineers should review for large datasets.   Spark automatically partitions RDD , but  engineers could consider change the partitioning scheme by changing the size of the partitions and number of partitions on case to case basis. 
Either too many small partitions  or too large partition will hinder the performance.  A single large partition will minimize parallelism resulting into  slow down of the overall processing, whereas , Many partitions will increase the overhead to manage the those partitions and additional data movement.


Once the data is partitioned , queries that matches the filter condition is only fetched from the I/O subsystem. This partition elimination capability  and ability to perform predicate pushdown greatly benefits the performance . 

What should be the partition size  and how many Partitions ?

	A. Select  partition column skewness and used frequently in the queries. 
	B. Select a column that ensures that number of partitions should be at least greater than the number of worker\compute nodes. 

One of the most common partitioning logic is Year. Quarter, Month , Days logic. Start with a higher granularity and reduce the scope as you traverse down. 




## File Size 

 In general, organize your data into larger sized files for better performance (256MB to 100GB in size). Some engines and applications might have trouble efficiently processing files that are greater than 100GB in size.
Sometimes, data pipelines have limited control over the raw data which has lots of small files. It is recommended to have  combine   that generates larger files to use for downstream applications.


## DataSet vs Dataframe 

Avoid RDD because it doesn’t have  query optimization through Catalyst. RDD APIs still exist in Spark 2.x for backwards compatibility, and should be used only in the scenarios that needs you to build a new custom RDD.

Dataframes is best choice for most of the workload that provides high performance  through catalyst but with no compile time checks or domain object programming support the familiar object-oriented programming style and compile-time type-safety of the RDD API 


Spark DataSet  API is very performant and suitable for complex ETL pipelines.  It provides optimization through catalyst and  supports compile time checks and domain object programming.
 Spark Dataset  gives the best of both Dataframes and RDDs.



## Coalesce vs Repartition 


Spark splits the data into partition and performs distributed computing on the each nodes. Repartition method can be used to either increase or decrease the number of partitions in a DataFrame.  However, when you use repartition , there is a shuffle operation that is performed and is an expensive operation. 

Whereas in coalesce method, the number of partitions is reduced in a dataframe. This performs consolidation without any shuffle operation. 

So, if you  need to repartition the data to less number of partition, then use coalesce and if you need more number of partition in subsequent dataframe, use repartition . 


## Explicit Schema 

If the schema  of the incoming data is fixed and conforms to a specific structure, it is always better to explicitly defines the schema. This minimizes the overhead of reading the  file to infer schema and then load the data to the dataset\dataframe. 

## MultiLine 

When the data is in a structured format such as CSV, using multiline=true  in the dataframe enables to read the data into a  single executor wherever possible, thus keeping the data local resulting into minimized shuffling and data  buffering\processing with complete usage of the executor and efficient memory usage.





Note: Delta capability will be captured in separate article. 

