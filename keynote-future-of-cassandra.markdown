#Keynote

###People who are using cassandra:
* Ooyla
* Netflix
* Twitter
* Rackspace

Overall trend: All analytics companies. Most people are not using it to
store their application data but rather using it to store and process
anayltics. 
Realtime layer of analytics. You also write the data to HDFS so that you
can run batch jobs on it later. Brisk changes some of this because you
can run hadoop and hive and pig jobs on the cassandra data. 

Definitely has some significant support from Datastax. Don't think the
project is going to die. 

###Datastax PR pitch

#### Things that are coming this year
* Performace improvements
* Manageability. Better experience for sys ops

#Keynote Talk (State of Cassandra)
* 100% increase in the number of people at the conference. 

* New feature integrated caching. 
  * Solves the cache coherence problem. You don't have to worry about
    having differences in data between the database and the cache
  * Moved the cache out of the the JVM. Cassandra 0.8 has native
    memmory cache. 

* Increased read performace. If you can keep the working set in
  memmory(this is not the whole dataset just the part you are using)

* No write locks at all. 
* No random writes to disk. Just streaming rights.

####Consistency
* You can tune cassandra to be a fully consistent system if you would
  like. You have a read-write quorum just like dynamo. 
    * This is a tradeoff of availability over consistency. If you have
      consistency then you must lose some availability(CAP theorum)
    
* There is now a local quorum. You can just check for consistency
  within a data center
  * there is some eventual consistency with other data centers. But
    you can reduce latency since you don't have to wait for external
    data center request

####Peformance improvements
* Multi data write progigation now only goes to 1 node in external data
  center. This node then propagates the data within the data center

* Automate the ability to create column families. You don't have to
  manually bring the nodes down. It is all automated. 

* Expiring columns
  * This will throw data away after a certain amojunt of time.
  * Now handled by the storage engine rather than a row scan.
    Performance improvment. 

#### Improved Indexes
* Secondary indexes on columns 

* CQL(Cassandra Query Languague)
  * Reduces the load on the clients. Much higher level interface than
    thrift. 
  * Allows for knowledge sharing between languages. There is a common
    interface. 

####Counters
  * There is a lot better counters now. Alows for cross data center
    counters.

####Other improvements
  * Fixed Memtable JVM tuning problem
  * Bulk load interface.
    * You can offload some of the work for bulk import from say hadoop
      to the clients. 


###Where is it Going

* 1.0 release in October. 

* CQL v 1.1
* Optimize reads for update heavy workloads

* Data compaction

####Data Repair

* Essentially Rsync for your data. 
* Goal is to minimize the amount of data that you have to send over the
  wire. 
* Fixe the 2 major problems with data repair in old versions


####CQL
* Adding Alter statement (Already has create drop)
* Counter support (You can update the counter just like in sql)
* TTL support
* Compound columns
* preparted statements.
* Moving away from thrift. You won't need thrift if you are using CQL
* Better range queries (You can cut down on amount of data that hadoop
  has to sift through using secondary indexes.)

####Brisk
* Allows for hadoop without HDFS. Builds file system on top of cassandra
  that is hadoop compliant. 
* Takes advantage of cassandras replication so you can move data from
  your realtime side to your batch layer. 
  * You don't want to run your batch analytics against your realtime
    nodes.

* You can use whatever encryption you want at the filesystem layer.
  There is support for encryption in the data interface layer. So you
  encrypt your data when it is moving around cassandra
