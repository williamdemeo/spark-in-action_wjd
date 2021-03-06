# Notes from the book Spark in Action

------------------------------------
# First Steps

## 1 Introduction to Apache Spark
### 1.5 Setting up the spark-in-action VM
#### 1.5.1 Downloading and starting the virtual machine

To run the VM requires a 64-bit OS with at least 3 GB of free memory and 15 GB of
free disk space.

We need to install these two software packages:

1. [Oracle VirtualBox](http://www.virtualbox.org) Oracle's
   free, open source hardware virtualization software.

   ___On Ubuntu Linux___ install with `sudo apt install virtualbox`

2. [Vagrant](http://www.vagrantup.com/downloads.html)
   HashiCorp's software for configuring portable development environments.

   ___On Ubuntu Linux___ install with `sudo apt install vagrant`

Next, create a folder for hosting the VM
(e.g. `spark-in-action_vm`) and enter it and download the
Vagrant box metadata JSON file:

    $ wget https://raw.githubusercontent.com/spark-in-action/first-edition/master/spark-in-action-box.json

Then download the VM itself:

    $ vagrant box add spark-in-action-box.json

The Vagrant box metadata JSON file points to the Vagrant box
file. The command will download the 5 GB VM box and register
it as the manning/spark-in-action Vagrant box. To use it,
initialize the Vagrant VM in the current directory by
issuing this command:

    $ vagrant init manning/spark-in-action

## 2 Spark fundamentals

### 2.1 Using the spark-in-action VM

**Start the spark-in-action VM**

    cd ~/git/Programming/spark-in-action_vm
    vagrant up

**Login to the VM**

As suggested in the book, I tried `vagrant ssh`.
I also tried `ssh 192.168.10.2`. Neither worked for me.
Here's what worked:

    ssh spark@192.168.10.2
    spark@192.168.10.2's password: spark

#### 2.1.1 Cloning the Spark in Action GitHub repository

Before doing anything else, clone our Spark in Action GitHub repository
into your home directory by issuing the following command (Git is already
installed in the VM ):

    $ git clone https://github.com/spark-in-action/first-edition

#### 2.1.2 Finding Java

**Locate the Spark and Java home directories on the spark-in-action VM**

    spark@spark-in-action:~$ export | grep SPARK
    declare -x SPARK_HOME="/usr/local/spark"

    spark@spark-in-action:~$ export | grep JAVA
    declare -x JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/jre"


#### 2.1.3 Using the VM's Hadoop Installation

    spark@spark-in-action:~$ hadoop fs -ls /user
    17/04/15 08:56:49 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    Found 1 items
    drwxr-xr-x   - spark supergroup          0 2016-04-19 18:49 /user/spark

The last command works because hadoop daemon is already running by default
on the VM, but if we had to start it ourselves we could with

    spark@spark-in-action:~$ /usr/local/hadoop/sbin/start-dfs.sh

We can stop it with

    spark@spark-in-action:~$ /usr/local/hadoop/sbin/stop-dfs.sh


#### 2.1.4 Examinin the VM's Spark installation

**Managing Spark releases**

    spark@spark-in-action:~$ ls /opt | grep spark
    spark-1.6.1-bin-hadoop2.6
    spark-2.0.0-bin-hadoop2.7

    spark@spark-in-action:~$ ls -la /usr/local/spark
    lrwxrwxrwx 1 root root 31 Sep 17  2016 /usr/local/spark -> /opt/spark-2.0.0-bin-hadoop2.7/

**Changing the Spark version**

Delete the link and create a new one; for example,

    sudo rm -rf /usr/local/spark
    sudo ln -s /opt/spark-1.6.1-bin-hadoop2.6 /usr/local/spark

### 2.2 Using Spark shell and writing your first Spark program

#### 2.2.1 Starting the Spark shell
Start the **spark shell** with the (obvious) command:

    spark@spark-in-action:~$ spark-shell
    Setting default log level to "WARN".
    To adjust logging level use sc.setLogLevel(newLevel).
    Spark context Web UI available at http://10.0.2.15:4040
    Spark context available as 'sc' (master = local[*], app id = local-1492247012310).
    Spark session available as 'spark'.
    Welcome to
              ____              __
             / __/__  ___ _____/ /__
            _\ \/ _ \/ _ `/ __/  '_/
           /___/ .__/\_,_/_/ /_/\_\   version 2.0.0
              /_/

    Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_72-internal)
    Type in expressions to have them evaluated.
    Type :help for more information.

    scala>

**Configure logging**

    nano /usr/local/spark/conf/log4j.properties

(The file was already there and already contained the suggested content.)


#### 2.2.2 The first Spark code example

Suppose we want to find out how many third-party libraries used by
Spark are licensed under the BSD license. Spark comes with a file named LICENSE in `$Spark_HOME` containing a list of all libraries used by
Spark and the licenses under which they're provided. Let's see how you
can ingest that file and count the lines using the Spark API:

    scala> val licLines = sc.textFile("/usr/local/spark/LICENSE")
    licLines: org.apache.spark.rdd.RDD[String] = /usr/local/spark/LICENSE MapPartitionsRDD[1] at textFile at <console>:24

    scala> licLines.filter(_.contains("BSD")).count // res0: Long = 33
    scala> licLines.filter(_.contains("BSD")).foreach(println)

In Scala, you don't always need the dots.  The following also works:

    scala> licLines filter(_.contains("BSD")) foreach(println)

#### 2.2.3 The notion of a resilient distributed dataset (RDD)

Although `licLines` and `bsdLines` feel and look like ordinary Scala collections (e.g., `filter` and `foreach` methods are available), they aren't. They're distributed collections, specific to Spark, called
resilient distributed datasets or *RDD*s.

The **RDD** is the fundamental abstraction in Spark. It represents a
collection of elements that is
+ Immutable (read-only)
+ Resilient (fault-tolerant)
+ Distributed (dataset spread out to more than one node)

### 2.3 Basic RDD Actions and Transformations

#### 2.3.1 Using the `map` transformation

#### 2.3.2 Using the `distinct` and `flatMap` transformations

```scala
spark@spark-in-action:~$ cat client-ids.log
15,16,20,20
77,80,94
94,98,16,31
31,15,20
spark@spark-in-action:~$ spark-shell

scala> val lines = sc.textFile("/home/spark/client-ids.log")
lines: org.apache.spark.rdd.RDD[String] = /home/spark/client-ids.log MapPartitionsRDD[1] at textFile at <console>:24

scala> val idsStr = lines.map(_.split(","))
idsStr: org.apache.spark.rdd.RDD[Array[String]] = MapPartitionsRDD[2] at map at <console>:26

scala> idsStr.first
res2: Array[String] = Array(15, 16, 20, 20)
```

So, if we read in a file containing

    15,16,20,20
    77,80,94
    94,98,16,31
    31,15,20

and make an RDD, the result will have `RDD[Array[String]]` type.
Each line in the file creates a single element of the RDD, and in this
case each element has type `Array[String]`.  For example, the first line
of the file above is read in as `Array(15, 16, 20, 20)`.

This can be seen by applying the `collect` action to the RDD as follows:

    scala> idsStr.collect
    res5: Array[Array[String]] = Array(Array(15, 16, 20, 20), Array(77, 80, 94), Array(94, 98, 16, 31), Array(31, 15, 20))

Note that the `split` function takes each input string to an array,
so applying `map split` to an `RDD[Array[String]]` results in an
array of arrays, i.e., `Array[Array[String]]` in the present case.
What if we want to concatenate all of the results into a single array.
Then we use `flatMap` instead of `map`.

    scala> val lines = sc.textFile("/home/spark/client-ids.log")
    scala> val idsStr = lines.flatMap(_.split(","))
    scala> idsStr.collect
    res0: Array[String] = Array(15, 16, 20, 20, 77, 80, 94, 94, 98, 16, 31, 31, 15, 20)


Our task was to find the number of unique clients who bought
anything, so we need the method `distinct` which returns a new
RDD with duplicates removed: `def distinct(): RDD[T]`

When called on an RDD, it creates a new RDD with unique elements (of the
same type, of course).

    scala> val uniqueIds = intIds.distinct
    uniqueIds: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[12] at distinct at <console>:27
    scala> uniqueIds.collect
    res15: Array[Int] = Array(16, 80, 98, 20, 94, 15, 77, 31)
    scala> val finalCount = uniqueIds.count
    finalCount: Long = 8

#### 2.3.3 Obtaining RDD's elements with the sample, take, and takeSample operations
Suppose you need to prepare a sample set that contains 30% of
client IDs randomly picked from the same log. The authors of
the RDD API anticipated this exact situation and implemented a
sample method in the RDD class. It's a transformation that creates
a new RDD with (a random number) of random elements from the calling `RDD` (`this`).

    def sample(withReplacement: Boolean, fraction: Double, seed: Long = Utils.random.nextLong): RDD[T]

+ `withReplacement` determines whether the same element may be
  sampled multiple times.

+ `fraction` determines the expected number of times each
  element is going to be sampled (as a number greater than zero),
  when replacement is used. When used without replacement, it
  determines the expected probability that each element is going to
  be sampled, expressed as a floating-point number between 0 and 1.

You needed to prepare a set of 30% (0.3) of all client IDs. You'll

**Sampling without replacement.**

    scala> val s = uniqueIds.sample(false, 0.3)
    s: org.apache.spark.rdd.RDD[String] = PartitionwiseSampledRDD[19] at sample at <console>:27
    scala> s.count
    res19: Long = 2
    scala> s.collect
    res20: Array[String] = Array(94, 21)

**Sampling with replacement.**
A 50% with replacement will make results more apparent:

    scala> val swr = uniqueIds.sample(true, 0.5)
    swr: org.apache.spark.rdd.RDD[String] = PartitionwiseSampledRDD[20] at sample at <console>:27
    scala> swr.count
    res21: Long = 5
    scala> swr.collect
    res22: Array[String] = Array(16, 80, 80, 20, 94)

**Deterministic sample size.**
If you want to sample an exact number of elements from an RDD, you
can use the `takeSample` *action*.

    def takeSample(withReplacement: Boolean, num: Int, seed: Long = Utils.random.nextLong): Array[T]

There are two differences between `sample` and `takeSample`.

1. `takeSample` takes an `Int` as its second parameter, which
   determines the number of sampled elements it returns. It always
   returns exactly `num` number of elements).
2. Whhereas `sample` is a transformation, takeSample is an action,
   which returns an array (like `collect`).

        scala> val taken = uniqueIds.takeSample(false, 5)
        taken: Array[String] = Array(80, 98, 77, 31, 15)

### 2.4 Double RDD functions

If you create an RDD containing only `Double` elements, several extra functions
become available, through the concept known as *implicit conversion*.

#### Scala's implicit conversion

Implicit conversion is a useful concept, heavily used in Spark, but it can be a
bit tricky to understand at first.

Let's say we have a Scala class defined like this:

    class ClassOne[T](val input: T) { }

`ClassOne` is type parameterized, so the argument input can be of type
`String`, `Int`, or any other type. Suppose we want objects of `ClassOne`
to have a method `duplicatedString()`, but only if input is a string,
and to have a method `duplicatedInt()` only if input is an integer.
You can accomplish this by creating two classes,
each containing one of these new methods. Additionally, you have to define two
implicit methods that will be used for conversion of ClassOne to these new
classes, like this:

    class ClassOneStr(val one: ClassOne[String]) {
    def duplicatedString() = one.input + one.input
    }
    class ClassOneInt(val one: ClassOne[Int]) {
    def duplicatedInt() = one.input.toString + one.input.toString
    }
    implicit def toStrMethods(one: ClassOne[String]) = new ClassOneStr(one)
    implicit def toIntMethods(one: ClassOne[Int]) = new ClassOneInt(one)

The compiler can now perform automatic conversion from type `ClassOne[String]`
to `ClassOneStr` and from `ClassOne[Int]` to `ClassOneInt`, and you can use
their methods on `ClassOne` objects. You can now perform something like this:

    scala> val oneStrTest = new ClassOne("test")
    oneStrTest: ClassOne[String] = ClassOne@516a4aef
    scala> val oneIntTest = new ClassOne(123)
    oneIntTest: ClassOne[Int] = ClassOne@f8caa36
    scala> oneStrTest.duplicatedString()
    res0: String = testtest
    scala> oneIntTest.duplicatedInt()
    res1: 123123


#### 2.4.1 Basic statistics with double RDD functions

Let's use the `intIds` RDD you created previously to illustrate the concepts in
this section. Although `intIds` contains  `Int` objects, they can be converted
to `Doubles`, so double RDD functions can be implicitly applied.
Using mean and sum is trivial:

    scala> val intIds = idsStr.map(_.toInt)
    scala> intIds.mean
    res0: Double = 44.785714285714285
    scala> intIds.sum
    res1: Double = 627.0

A bit more involved is the stats action. It calculates the count and sum of all
elements; their mean, maximum, and minimum values; and their variance and
standard deviation, all in one pass, and returns an `org.apache.spark.util.StatCounter`
object that has methods for accessing all these metrics.
`variance` and `stdev` actions are just shortcuts for calling
`stats().variance` and `stats().stdev`:

    scala> intIds.variance
    res2: Double = 1114.8826530612246
    scala> intIds.stdev
    res3: Double = 33.38985853610681

#### 2.4.2 Visualizing data distribution with histograms
#### 2.4.3 Approximate sum and mean

### 2.5 Summary

------------------------------------------

## 3 Writing Spark applications
### 3.1 Generating a new Spark project in Eclipse
### 3.2 Developing the application

#### 3.2.1 Preparing the GitHub archive dataset
#### 3.2.2 Loading JSON
#### 3.2.3 Running the application from Eclipse
#### 3.2.4 Aggregating the data
#### 3.2.5 Excluding non-employees
#### 3.2.6 Broadcast variables
#### 3.2.7 Using the entire dataset

### 3.3 Submitting the application
#### 3.3.1 Building the uberjar
#### 3.3.2 Adapting the application
#### 3.3.3 Using spark-submit


### 3.4 Summary

------------------------------------------

## 4 The Spark API in depth

### 4.1 Working with pair RDDs
#### 4.1.1 Creating pair RDDs
#### 4.1.2 Basic pair RDD functions

### 4.2 Understanding data partitioning and reducing data shuffling

#### 4.2.1 Using Spark's data partitioners
#### 4.2.2 Understanding and avoiding unnecessary shuffling
#### 4.2.3 Repartitioning RDDs
#### 4.2.4 Mapping data in partitions

### 4.3 Joining, sorting, and grouping data
#### 4.3.1 Joining data
#### 4.3.2 Sorting data
#### 4.3.3 Grouping data

### 4.4 Understanding RDD dependencies
#### 4.4.1 RDD dependencies and Spark execution
#### 4.4.2 Spark stages and tasks
#### 4.4.3 Saving the RDD lineage with checkpointing

### 4.5 Using accumulators and broadcast variables to communicate with Spark executors
#### 4.5.1 Obtaining data from executors with accumulators
#### 4.5.2 Sending data to executors using broadcast variables

### 4.6 Summary

------------------------------------------

# MEET THE SPARK FAMILY
## 5 Sparkling queries with Spark SQL
## 6 Ingesting data with Spark Streaming
## 7 Getting smart with MLlib
## 8 ML: classification and clustering
## 9 Connecting the dots with GraphX

---------------------------------------------

# SPARK OPS
## 10 Running Spark
## 11 Running on a Spark standalone cluster
## 12 Running on YARN and Mesos

--------------------------------------------

# BRINGING IT TOGETHER
## 11 Case study: real-time dashboard
## 12 Deep learning on Spark with H2O
