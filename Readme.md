
# Cloud Computing Mini Project 3 - Queries


Preprocess data: split PATH Column into 3 columns: METHOD, PATH and PROTOCOL
(Python notebook with data preprocessing steps attached)


# PART 2


## Step 1 - start Cassandra on all 3 VMs using command: sudo cassandra -Rf
## Step 2 - start CQL shell: cqlsh CC-MON-3

### Create Keyspace

CREATE KEYSPACE online_accesslog_ks WITH replication = { 'class': 'SimpleStrategy', 'replication_factor' : 2};


### Create table

CREATE TABLE online_accesslog_ks.table1 (
ID int,
IP text,
PATH text,
PRIMARY KEY((ID), IP, PATH)
);


### Import Data from CSV

COPY online_accesslog_ks.table1 (ID,IP,PATH) FROM '/home/student/Mini_proj_3/access_logs_paths.csv' WITH  DELIMITER= ',' AND HEADER = TRUE and NULL = 'null';

### CREATE INDEX

CREATE INDEX ip_index ON online_accesslog_ks.table1 (IP);

CREATE INDEX path_index ON online_accesslog_ks.table1(PATH);



# PART 3



### Part 3.1: How many hits were made to the website item “/assets/img/release-schedule- logo.png”?

SELECT COUNT(*)
	FROM online_accesslog_ks.table1
	WHERE PATH = '/assets/img/release-schedule-logo.png'
	ALLOW FILTERING;

 

### Part 3.2: How many hits were made from the IP: 10.207.188.188?

SELECT COUNT(*)
	FROM online_accesslog_ks.table1
	WHERE IP = '10.207.188.188'
	ALLOW FILTERING;



## Part 3.3 and 3.4



#### Queries:


User-defined function 1:


CREATE OR REPLACE FUNCTION online_accesslog_ks.input_count(state map<text, int>, path text)
CALLED ON NULL INPUT
RETURNS map<text, int>
LANGUAGE java
AS $$
Integer count = (Integer) state.get(path);
if (count == null)
                count = 1;
else
                count++;
state.put(path, count);
return state;
$$;




User-defined function 2:

CREATE OR REPLACE FUNCTION online_accesslog_ks.find_max_count(input map<text, int>) 
RETURNS NULL ON NULL INPUT
RETURNS map<text, int>
LANGUAGE java
AS $$
Integer max_value = Integer.MIN_VALUE;
String data="";
for (String temp1 : input.keySet())
{
        Integer temp2 = input.get(temp1);
        if (temp2 > max_value)
        {
             max_value = temp2;
             data = temp1;
        }
}
Map<String,Integer> max_map = new HashMap<String,Integer>();
max_map.put(data,max_value);
return max_map;
$$;




Aggregate function 3:

CREATE OR REPLACE AGGREGATE max_hits(text) 
SFUNC input_count 
STYPE map<text, int> 
FINALFUNC find_max_count
INITCOND {};


### Queries For Results:


3.3. Which path in the website has been hit most? How many hits were made to the path?

select max_hits(path) from table1 allow filtering;

3.4. Which IP accesses the website most? How many accesses were made by it?

select max_hits(ip) from table1 allow filtering;
