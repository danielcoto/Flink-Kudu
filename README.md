# Flink-Kudu
This is a simple PoC for integrating Apache Kudu and Apache Flink. Main goal is to be able to read/write data from/to Kudu using the DataSet and DataStream APIs.
Data flows patterns:
* Batch
  * Kudu -> DataSet\<RowSerializable\> -> Kudu
  * Kudu -> DataSet\<RowSerializable\> -> other source
  * Other source -> DataSet\<RowSerializable\> -> other source
* Stream
  * Other source -> DataStream \<RowSerializable\> -> Kudu


```java

/* batch mode */
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

KuduInputFormat ksource = new KuduInputFormat(SOURCE_TABLE, KUDU_MASTER);
KuduTypeInformation typeInformation = new KuduTypeInformation(new RowSerializable());

DataSet<RowSerializable> input = env.createInput(ksource, typeInformation);

input.output(new KuduOutputFormat(KUDU_MASTER, DEST_TABLE, columnNames, KuduOutputFormat.CREATE));
```

```java

/* Streaming mode */
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<String> stream = env.fromElements("data1 data2 data3");
DataStream<RowSerializable> stream2 = stream.map(new MapToRowSerializable());

stream2.addSink(new KuduSink(KUDU_MASTER, DEST_TABLE, columnNames));

```



## Requirements

* Flink and Kudu compatible OS
* Apache Flink (version 1.1.3)
* Apache Kudu (version 1.1.0)
* Maven (version 3.3.9)
* Java (Version 8)


## Work in Progress
### Completed
- [x] Sink implementation for Batch Processing and example job.
- [x] Sink implementation for Streaming Processing and example job.
- [x] Source implementation for Batch Processing and example job.

### Working
- [ ] Unit Tests for main classes.
- [ ] Kafka integration in example job

# Build library

```
git clone https://github.com/rubencasado/Flink-Kudu.git
cd Flink-Kudu
mvn clean install -DskipTests 
mvn package -DskipTests
```
Generated JAR will be located at "*Flink-Kudu / target / flink-kudu-1.0-SNAPSHOT.jar*"

# Execution

```
<Flink-instalation-folder>/bin/flink run -c <Job-package-path> target/flink-kudu-1.0-SNAPSHOT.jar param1 param2 ...
```
For example:
```
/opt/flink-1.1.3/bin/flink run -c es.accenture.flink.Job.JobBatchSink target/flink-kudu-1.0-SNAPSHOT.jar mytable CREATE localhost
```
**Included examples**
- [x] JobSource (**es.accenture.flink.Job.JobSource**): Reads data from a Kudu table and prints the information (*DataSet API, Batch Mode*).
- [x] JobBatchSink (**es.accenture.flink.Job.JobBatchSink**): Saves data from a DataSet into a Kudu table(*DataSet API, Batch Mode*). Input parameters: *table_name*, *table_mode* (CREATE, APPEND or OVERRIDE) and *host*.
- [x] JobBatchInputOutput (**es.accenture.flink.Job.JobBatchInputOutput**): Reads data from a Kudu table, executes some basic operations using DataSet API and save the result into a Kudu table(*DataSet API, Batch Mode*).
- [x] JobStreamingSink (**es.accenture.flink.Job.JobStreamingSink**): Saves data from a DataStream into a Kudu table (*DataStream API, Streaming Mode*). Input parameters: *table_name* and *host*.
- [ ] JobStreamingInputOutput (**es.accenture.flink.Job.JobStreamingInputOutput**): Reads data from Kafka, executes some basic operations using DataStream API and saves results into a Kudu table (*DataStream API, Streaming Mode*).
