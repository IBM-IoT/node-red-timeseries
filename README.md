node-red-contrib-timeseries
========================
A [Node-RED](http://nodered.org) node to output to and read from a TimeSeries database.

Install
-------
Run the following command in the root directory of your Node-RED install:

```npm install node-red-contrib-timeseries```

Once installed, the TimeSeries input and output nodes will be available in the 
node pallet.
    
    
TimeSeries Database Requirements
------
You must have a TimeSeries database running with the wire listener installed
and running in REST mode with SQL passthrough turned on.

See the following documentation for information regarding the setup and configuration of the wire listener:

http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.json.doc/ids_json_004.htm


Overview
-----
Two nodes are included in this install. The TimeSeries output node and the TimeSeries input node.

TimeSeries output node
-----

The output node writes data to the TimeSeries database. It takes in as input a JSON string from the incoming ```msg.payload``` and makes a REST POST call to insert that JSON as a row into the TimeSeries database.
The new row is inserted into a virtual table (VTI) created off of the base table specified in the node configuration. The virtual table, which 
is given the name ```<$BASE_TABLE_NAME>_v```, is automatically created on node deployment or when Node-RED starts.

For example, if you provide a base TimeSeries table called ```sensors```, a virtual table called ```sensors_v``` will automatically be created.

Both regular and irregular TimeSeries entries can be made.

See the following documentation for more information about the TimeSeries REST API:

http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.json.doc/ids_json_043.htm

Further information regarding TimeSeries virtual tables can be found here:

http://www-01.ibm.com/support/knowledgecenter/SSGU8G_11.70.0/com.ibm.tms.doc/ids_tms_099.htm

#### Examples:
**Regular Time Series**
The following JSON would add a regular TimeSeries entry of value "65" to sensor with ID "54".
```
{"sensor_id": 54,
"measure_unit": "F",
"value": 65 }
```

**Irregular Time Series**
The following JSON would add data to the ```json_data``` field (which in this case, is itself JSON/BSON data) to an irregular TimeSeries entry for sensor with ID "1" at time 2015-03-11 16:38:40.
```
{"id":1,
"tstamp":"2015-03-11 16:38:40.00000",
"json_data":{"nodeId":2,
 "commandClass":50,
"label":"Current",
"max":0,"min":0,
"units":"A",
"value":0}}
```

#### Configuration

Double click on the TimeSeries output node to open the node configuration.
The following configurations can be set for the output node.

**Server** - Specify the TimeSeries database server to connect to.
Note that any server configuration you make can be shared across any number
of input or output nodes.
    
* Host - Hostname of the database server.
* Port - Port must be the same used by the wire listener.
* Database - The name of the database to connect to.
* Username - (Only required if the server requires authentication)
* Password - (Only required if the server requires authentication)
* Name - (Optional label for this database connection.)
    
**Table** - The TimeSeries table to insert data into. Note that a
corresponding Virtual table (VTI) will automatically be created for this base
table.

**Name** - An optional label for this node.


TimeSeries input node
-----

The input node performs the specified statistical aggregation on TimeSeries data
over a specified time range.

The result is returned in ```msg.payload```.

This node provides the functionality of the TimeSeries ```AggregateBy()``` function

See the IBM Knowledge Center (http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.tms.doc/ids_tms_141.htm)
for more information on the AggregateBy() function.

Takes in a JSON object which specifies the row ID to aggregate on and optionally the timestamp on which to start aggration.

For example, if the unique key column is named "id" and we want to aggregate on row with ID "eca86bf831dd.ZWnode3" we would pass in the JSON:
{id:"eca86bf831dd.ZWnode3"}
and specify that the unique key column is name "id" in the Name of unique key field in incoming JSON field."

#### Configuration

Double click on the TimeSeries input node to open the node configuration.
The following configurations can be set for the input node.

**Server** - Specify the TimeSeries database server to connect to.
Same as in the output node.
Note that any server configuration you make can be shared across any number
of input or output nodes.
    
* Host - Hostname of the database server.
* Port - Port must be the same used by the wire listener.
* Database - The name of the databse to connect to.
* Username - (Only required if the server requires authentication)
* Password - (Only required if the server requires authentication)
* Name - (Optional label for this database connection.)


**Table** - The base TimeSeries table that has data you want to aggregate on.

**TS Column** - The name of the table's TimeSeries column.

**Fields to aggregate** - Select the field(s) to peform aggregation on and the
type of aggregration you want to do.

Press ```New Aggreation``` to add a new field. Select the new aggregation type
to perform and enter the field name. Dot notation is supported for JSON values.

The following statisitcal aggregations are supported.

* Avg
* Sum
* Min
* Max

There is no limit to the number of fields you can aggregate.

**Name of unique key field in incoming JSON** -

Specify the name of the unique key field for the TimeSeries table. This value
will be used to find the unique key value in the incoming JSON.

For example, if the unique key column is named "id" and we want to aggregate on
row with ID "eca86" we would pass in JSON containing the line:

```
{ ...
id: "eca86"
... }
```

and specify that the unique key column has name "id" in the
"Name of unique key field in incoming JSON field."

Only one ID value can be specified in the incoming JSON>


**Aggregration mode** -

* Discrete - The value we are aggregating is a discrete value and there is no intermittent 
values between the stored values.

Stock purchase orders are an example of discrete values.

* Continuous - The value we are aggregrating has intermittent values between the actual recorded values.

For example, temperature values have intermittent values even when values are not being recorded.

If a temperature sensor records a value of 50 degress at 
12:00 and and a value of 100 degrees at 12:14, we will get a value of 75 degrees
for when we try to get the get the average temperature between 12:00-12:15 in discrete mode.
However, this value is incorrect because what we really want to average is 13 minutes
of 50 degrees and 2 minutes of 100 degrees. The continuous mode automatically takes this into account
and automatically "weights" the values to take into account the implicit, interrmittant values.


**Aggregate Between** -

Select the range of time on which to aggregate on.

* now - Between the current clock time and however long ago in the past.

* timestamp - Between the unix timestamp passed in on the incoming JSON payload and however long in the past. Use the
 ```timestamp``` field in the incoming JSON to specify the Unix timestamp. For example:

```
{ ...
"timestamp": 1427399297364 # Unix timestamp
... }
```

**Calendar pattern length** -

The size of the calendar pattern on which to aggregate.

For most uses, this will be the same as your "aggregate between" value.

**Name** - An optional label for this node.
