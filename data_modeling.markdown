# Data Modeling with Cassandra

### Data Modeling Goals
* You want to start with what you want to get then work backwards to how
  you should store the data
* You want to store related data close together on disk.
* Usually better to keep a record/journal of things that happen rather
  than updating a value.

### Click Stream Data Example
* All the data about how a user interacts with a site

* 1 CF for the sessions a user has 
  * basically { "user_id" => {"session_id(timeuuid)" => <empty or
    aggregated data>}} 
  * manual index
  * lets you see the most recent session and find any timeframe that you
    want

* 1 CF for the actual sessions data
  * { "sessionid(timeuuid)": { "timestamp_01" => <click data(json/xml/bson)> }}
  * very easy to page the data so you can see the 10 most recent for
    instance
  * again don't use super columns

* If you don't want to store aggregated data in user_session CF then you
  can use secondary index on session_data and not use user_session

* Be careful of making the rows to wide. Could require more than 1 disk
  read

* Lots of requests for a single row can results for hotspots. Just
  adjust how large you want to timespace the rows based on how much data
  and activity you have. 

* if you have data that you know won't be added to you can just
  aggregate it once then keep the aggregated values.
  * I.E. All activity in January won't get any more writes after January
    ends. Therefore you can aggregate it then store that aggregate. 
    * Kind of like a permament cache. 

* if you know you will never want data over a certain age. Set a TTL on
  the data. More effecient than manually scanning later. Also helps keep
  size down 

#### Storing Time Data With Time Zones
* To store time data. You want to have the granularity that you need for
  time zones. You can probably get away with hourly but sometimes you
  need 15min slices or 30min slices
* To get 7 days from now. You calculate all day columns that are totalyy
  withing your time zone. Then go down to hourly granularity to figure
  out the amount of data withing those days that are on the fence of the
  time zone

### Dealing with Consistency
* You should not try and get full ACID. It will be a hug PITA
* You can use transaction logs to get some level of consistency and
  transactions
  * You write out what you are going to do before you do it.
    * You can use a CF to store the transaction log
  * Then you perform the actions in order
  * Delete to row that stores the actions
  * If one of the actions fail use some revert log to replay the actions
    that have already been preformed.

* Write to transaction log in quorum or local quorum
  
* If you are going to use logs use short GC_Grace just for the CF that
  stores the transaction log. This prevents tombstones. 
    * Most of the transactions should be tombstones since you write them
      very quickly 

####Handling Failures in transactions

* If you fail before it starts then don't worry it isn't your problem
  nothing touched the database

* After insert into transaction log but before actions. Just delete the
  transaction entry
* After insert in the middle of actions. You need to replay the actions
  so that you can regain your consistency. 
    * Usually this means writing back the old data so that it has a new
     timestamp and cassandra will just ignore the bad data that we wrote

* If you fail after the actions. Then the data is consistent but the
  client doesn't know that. You can have cron jobs that come through and
  grab all the transactions in a time slice and replay all that data. 
    * This will waste a bit of cpu but not be a huge deal

* Doesn't work for counters :( 

* You need to deal with some temporary inconsistencies between the time
  some of the actions were performed and the time the transaction log is
  replayed
    * This is the same for any distributed system even mySQL
    * You can tune how log this is by tuning how often you want to query
      the transaction log and replay changes

* To avoid polling the transaction log you can use a message queue.
  Write to the queue before you replay the transaction log after the
  failure occurs
