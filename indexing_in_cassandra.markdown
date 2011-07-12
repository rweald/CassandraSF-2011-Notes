#Indexes in Cassandra

###Background
* There were no built in secondary indexs until 0.6
* Forcing you to build custom indexes lets you get a great knowledge of
  the whole system
* Don't use supercolumns :)

#### Built in secondary indexes
* Not the same as SQL indexes. 
* You don't understand all the things that cassandra is good at if you
  are just using the built in indexes. 
* There are advanced techniques that let you extend the built in
  secondary indexes. 

###Quick Review
* Primary index(Row keys sql analogy)
* Get vs Find. Row keys is really good for gets. Not good for finds. 

* use alternate indexes to do finds


##Alternate Indexes

* native secondary indexes
* wide row as lookup and grouping tables
* custom secondary indexes.
* inverted indexes

* There is no official way to do indexes. There are just preferences and
  multiple ways to accomplish what 

####Native secondary indexes
* Look like sql indexes, especially when using cql
  * They are not like sql indexes. 
* They are bulding hidden column family for each index.
* Nodes index the rows they store. 
* These are a great starting point. Use them until you need something
  more complex then move on

* Some Limitations
  * Not good from high cardinality values. 
  * Requires at least 1 equality comparisoin in a query. 
  * Unsorted. Results are in tokan order not query value order
  * You have to store the data in your rows in a format that cassandra
    understands

####Custom secondary indexes
* It is not as hard as you think. You just have to look at cassandras
  unique features. 

*Wide Rows
  * Basis of all indexing, organizing, and relationships in cassandra
  * If your data model has no rows with over 100 columns you are doing
    it wrong, or you shouldn't be using cassandra
  * several million rows is common

* Conventional rows as records

* You can use wide rows for grouping
* wide rows are a simple form of index.

* CF as indexes
  * Columns store things really fast. 
  * can be sliced and returend from range
  * If a target key timeUUID you get both grouping and sort by timestamp
  * Best option when you need to combine groups, sorts, and search

* CF indexes only work on 1-1 relationships.

* Super CFs should be used with caution. 
  * Not officially deprected but not really used and not liked.
  * Sorts only work on supercolumn. 
  * You are at the max for nesting. You can't get any more nesting. 
  * Most projects have moved away from supercolumns

* You can get around the uniqueness constraint of column names by using
  composite keys names.
  * EX// {"anderson", 1} and { "anderson", 2}
  * These composite columns names now have native support
  * Each of the components can be any of the types that cassandra
    recognizes.(You can not nest components in components)
  
* Two composite types in cassandra
  * Main difference for use in indexes is whether you need 1 CF per
    index vs one CF for all indexes with one row per index.

* Static composites
  * fixed number and order. 
  * definded in column configuration

* Dynamic composites
  * any # and order of types at runtime. 

* Typical index from a composite
  1) Terms to query on i.e. first_name
  2) id
  3) Timestamp

* queries are easy. regular column slice operations
* Updates are harder
  * Need to remove old value and insert the new value
  * Uh oh, read before write??
  
* Example User by Location
  * 3 CFs not 2
  * first 2 CFS
    1) Users
    2) Indexes
      * Composite key: < location, user_key, timestamp> 
    3) Users_index_entries
      * When you update your main user data column you will put it here
        with a timestamp. This is where you read from the prevent
        concurrency problems
      * You can quickly get all the old values

  * If you only use 2 CFs you will have problems with race conditions
    and you will need additonal locking abilities. 

  * Updating the index
    * read previous index values from the user_index_entities CF
    * delete them from the user_index_entities CF
    * store the new ones in the user_index_entities
    * finally store the actual data in the user_column

* You don't need locking the index will be eventually consistent.
* You can rerun the opperation so you don't have to 
* You can get a false positive. because index is eventually consistent. 
  * You can filter on reads so that you don't display false to users

* include additional denormalized values in the index for faster lookup
* use composites for column values too not just column names

* Custom indexes allow you to index into json that you store in the data
  column. This gives you document like functionality

* Background reading
  * anuff.com/2011/02/indexing-in-cassandra.html
  * anuff.com/2011/02/secondary-indexes-in-cassandra.html
usergrid.com
