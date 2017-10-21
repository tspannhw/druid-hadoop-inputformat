## Druid Hadoop InputFormat

This is a Hadoop InputFormat that can be used to load Druid data from deep storage.

### Installation

To install this library, run `mvn install`. You can then include it in projects with Maven by using the dependency:

```xml
<dependency>
  <groupId>io.imply</groupId>
  <artifactId>druid-hadoop-inputformat</artifactId>
  <version>0.1-SNAPSHOT</version>
</dependency>
```

### Example

Java example of how to directly read Druid deep storage into a Spark RDD:

```java
final JobConf jobConf = new JobConf();
final String coordinatorHost = "localhost:8081";
final String dataSource = "wikiticker";
final List<Interval> intervals = null; // null to include all time
final DimFilter filter = null; // null to include all rows
final List<String> columns = null; // null to include all columns

DruidInputFormat.setInputs(
    jobConf,
    coordinatorHost,
    dataSource,
    intervals,
    filter,
    columns
);

final JavaPairRDD<NullWritable, InputRow> rdd = jsc.newAPIHadoopRDD(
    jobConf,
    DruidInputFormat.class,
    NullWritable.class,
    InputRow.class
);
```


Scala example of how to read directly from Druid deep storage, into an RDD, and convert the RDD into a Dataframe:

```bash
spark-shell --jars target/druid-hadoop-inputformat-0.1-SNAPSHOT.jar
```

```scala
import spark.implicits._
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._
import org.apache.hadoop.mapred.JobConf
import io.druid.data.input.InputRow
import io.imply.druid.hadoop.DruidInputFormat
import org.apache.hadoop.io.NullWritable

val jobConf = new JobConf();
val coordHost = "localhost:8081"
val dataSource = "data_minute"
val interval = null      // Type: List<Interval> // null gets all intervals
val filter = null        // Type: DimFilter      // null passes no filters
val columns = null       // Type: List<String>   // null gets all columns

DruidInputFormat.setInputs(jobConf,coordHost,dataSource,null,null,null)

val dataDF = sc.newAPIHadoopRDD(jobConf, classOf[DruidInputFormat], classOf[NullWritable], classOf[InputRow]).map{x => x._2}.map{x => (x.getTimestamp().toString(),x.getTimestampFromEpoch(),x.getDimension("function").get(0),x.getFloatMetric("value_sum"))}.toDF("timestring","epoch","tag","sum").select(date_format($"timestring", "yyyy-MM-dd'T'hh:mm:ss:SSS'Z'") as("time"),$"epoch",$"tag",$"sum")

dataDF.show
```