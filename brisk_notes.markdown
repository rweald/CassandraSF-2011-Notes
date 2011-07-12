#Counters in Cassandra

###Prior to 0.8.0

* You have to use some external locking mechanizm. i.e. zookeeper
* use a column per increment then have to sum all those columns
* use another database (redis, postregsql)

Overall you were kind of screwed on performance and developer
friendliness

### After 0.8.0

* You have a native counter column type in cassandra. Fast read/write. 
* distributed design. No SPOF
* You have to make sure you set distribute on write to true. Otherwise
  you will not be able to perform r/w with quorum above 1

* You get multi-datacenter support automatically since it is built as a
  CF. 


##How it is not implemented?

* You don't treat it like a traditional write. Because you can have
  problems with the hinted handoff and data repair if the nodes fail.
  * You can get counters that are off. 
  * you can not detect that there was supposed to be an increment that
    you missed. This means that the system is convinced that it is right
    but in reality the count is wrong.

* Even with vector clocks you can not necessarily guarantee that the
  value is accurate. 
  * the vector clock will tell the database that there is a problem but
    it can not tell you what is wrong and what the value should be

###How it is implemented on the cluster side
* Partitioned counters
  * You store the value along with a version in a vector. Essentially it
    is a vector clock plus a value that tells you the counter at each
    node update
    * EX// [value(version), value(version), value(version)]
  * This allows you to essentially replay increments on nodes after they
    rejoin the ring.

  * To coordinate between data centers and replicas the increment is
    just sent to 1 node. That node then propagates the change through
    the rest of the relavant nodes in the data center

### How it is implemented on the individual node
* There are 2 types of request that the node can recieve.

1) An increment is requested by a client.
  * There is never any read. You just push an increment of the value as
    well as the version in the vector. 

2) Increment is propagated to the node as per the r/w quorum
  * There is a read in this case. You must read the current vector from
    this node.
  * Compare the node vector with the vector that is passed in the rpc
    call. 
  * sum the two up so that they are consistent then write the new value

* You do not have to lock the db ever to perform the increment. The
  vector algorithms allows us to just send the most recent version if we
  simultaneaus reads and writes.
  * This value might not be 100% consistent depending on how you set the
    quorum.
  * Allows it to be really fast because no locks is a good thing
  * There is one catch. If you are the source node you can discard values
    in the vector that are passed to yourself. Since you are yourself
    there is no way there can be a more recent version than me.
    Therefore we just return the most recent write.

  * If you set the quorum to 1 then you are basically just pushing into
    memmory because any reads will happen in the background and the
    latency will not be 

### Limitations of the counters
1) If a write fails due to external error (RequestTimeout)
   the cassandra node internally does not know if the write was
   persisted. This means you can have overcounting

2) You can not remove a counter and then try and reincrement it. If you
   do the behavior is undefined. To reset a counter do an insert value
   to the starting value

3) If you lose a SSTable then you have to remove all SSTables in the CF
   then repair

4) No TTL support for counters

5) No secondary indexed for counter columns

6) Only 1 full CF of counters so far. That means you can't mix counters
   with other data types in a CF

7) No CLANY
