# How to industrialize a Hive data production chain

## Zeppelin notebooks

We developed Spark Scala code on a [Zeppelin](https://zeppelin.apache.org/) notebook to be able to transform raw data into results.

To make quick checks, we have a [Hue](http://gethue.com/) interface to execute direct SQL transactions from Hive.

## Versioning the notebook

We started versioning the notebook codes in git to ensure we don't lose anything.

Raw data is updated everyday and we have new features to implement.

To deploy new output data, we launch manually the notebook.

## Splitting the notebook into smaller steps

To be able to debug and test features faster, we split the notebook into multiple coherent notebooks that roughly look like:

* Cleaning
* Computation
* Export output data to an Elasticsearch index

Each notebook produce an intermediary result stored in a table

## Use Maven + Oozie to be independent from Zeppelin

To collaborate easily and be independent from Zeppelin interface, we created a Scala Maven project with our code versioned with git.

The project is split into multiple modules following our notebooks.

To deploy the code, we package it with maven, and upload the jar archives to HDFS.

To run the code, we created [Oozie](http://oozie.apache.org/) workflows that use our different archives.

## Use different databases to test before deploying

We created a snapshot of the database and other Oozie Workflows that are not linked to our production data:

* If the workflow fails or produce wrong data, there is no impact on production
* The snapshot is not updated daily so we know what results to expect on our different steps and know if our modifications break something.

## Use coordinators to automatically run our algorithms

To run automatically the code everyday when we receive new data, we created an Oozie Coordinator that use our workflows

## Test locally

We used [Hadoop Unit](https://github.com/jetoile/hadoop-unit) to write unit tests for our algorithms, so we can develop without deploying to the cluster.

We split our code to be able to isolate the spark conf and the side effects in the tests:

```scala
// CleaningMain.scala
package com.company.project.cleaning

import org.apache.spark.SparkConf

object CleaningMain {
def main(args: Array[String]): Unit = {
val conf = new SparkConf()
.setAppName("project cleaning")
.setMaster("yarn-cluster")
val job = new Cleaning(conf)
val data = job.importData()
val cleanedData = job.run(data)
job.createOutputTable(cleanedData)
job.shutdown()
}
}

// Cleaning.scala
package com.company.project.cleaning

import org.apache.log4j.{LogManager, Logger}
import org.apache.spark.sql.hive.HiveContext
import org.apache.spark.sql.DataFrame
import org.apache.spark.{SparkConf, SparkContext}

class Cleaning(conf: SparkConf) {
val log: Logger = LogManager.getLogger("cleaning")
val sc = new SparkContext(conf)
val sqlContext = new HiveContext(sc)

def importData(): DataFrame = {
...
data
}

def createOutputTable(cleanedData: DataFrame): Unit = {
cleanedData.registerTempTable("temp_cleaned_data")
sqlContext.sql(s"DROP table database.clean_data")
sqlContext.sql(s"CREATE table database.clean_data STORED AS ORC AS SELECT * FROM temp_cleaned_data")
log.info("wrote data table")
}

def run(data: DataFrame): DataFrame = {
log.info("Starting cleaning processing...")
val cleanedData = processData(...)
log.info("Finished cleaning processing")
cleanedData
}

def shutdown(): Unit = {
if (sc != null) {
sc.stop()
}
}

def importCsvFile(path: String): DataFrame = {
sqlContext.read.format("com.databricks.spark.csv")
.option("header", "true")
.option("nullValue", "null")
.option("delimiter", ",")
.load(path)
}
}

//CleaningTest.scala
package com.company.project.cleaning

import fr.jetoile.hadoopunit.HadoopUnitConfig
import fr.jetoile.hadoopunit.HadoopUnitConfig._
import fr.jetoile.hadoopunit.test.hdfs.HdfsUtils
import org.apache.commons.configuration.{Configuration, PropertiesConfiguration}
import org.apache.hadoop.fs.Path
import org.apache.spark.SparkConf
import org.assertj.core.api.Assertions._
import org.scalatest._
import org.apache.spark.sql.functions.col


class CleaningTest extends FeatureSpec with BeforeAndAfterAll with BeforeAndAfter with GivenWhenThen {

var configuration: Configuration = _
val inputCsvPath: String = "/input/csv"

override protected def beforeAll(): Unit = {
configuration = new PropertiesConfiguration(HadoopUnitConfig.DEFAULT_PROPS_FILE)
}

before {
val fileSystem = HdfsUtils.INSTANCE.getFileSystem

val hdfsPath = "hdfs://" + configuration.getString(HDFS_NAMENODE_HOST_KEY) + ":" + configuration.getInt(HDFS_NAMENODE_PORT_KEY) + "/"

fileSystem.copyFromLocalFile(new Path(CleaningTest.this.getClass.getClassLoader.getResource("cleaning_input.csv").toURI),
new Path(hdfsPath + inputCsvPath + "/cleaning_input.csv"))
fileSystem.copyFromLocalFile(new Path(CleaningTest.this.getClass.getClassLoader.getResource("cleaning_expected_output.csv").toURI),
new Path(hdfsPath + inputCsvPath + "/cleaning_expected_output.csv"))
}

after {
HdfsUtils.INSTANCE.getFileSystem().delete(new Path("/input"))
}

feature("Cleaning test") {
scenario("Correctly cleaned data") {

Given("a local spark conf")
val conf = new SparkConf()
.setAppName("init")
.setMaster("local[*]")
.set("spark.driver.allowMultipleContexts", "true")

And("my job")
val job = new Cleaning(conf)

When("I read the csv files")
val hdfsPath = "hdfs://" + configuration.getString(HDFS_NAMENODE_HOST_KEY) + ":" + configuration.getInt(HDFS_NAMENODE_PORT_KEY) + "/"
val importData = job.importCsvFile(hdfsPath + inputCsvPath + "/cleaning_input.csv")
val expectedCleanData = job.importCsvFile(hdfsPath + inputCsvPath + "/cleaning_expected_output.csv")

Then("I have the right cleaned lines")

val cleanedData = job.run(importData)
val orderedColumns = expectedCleanData.columns.sorted.map(name => col(name))
assertThat(expectedCleanData.select(orderedColumns:_*).except(cleanedData.select(orderedColumns:_*)).count()).isEqualTo(0)
assertThat(cleanedData.select(orderedColumns:_*).except(expectedCleanData.select(orderedColumns:_*)).count()).isEqualTo(0)

job.shutdown()
}
}
}

```

## Version and deploy Oozie workflows

TODO

## Future

Continuous deployment

Continuous integration

