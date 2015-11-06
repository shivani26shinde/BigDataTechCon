# Flume, Kafka + Camus Tutorial

[Tutorial Requirements](https://github.com/mbkeane/BigDataTechCon/wiki/Tutorial-Requirements)

[Anatomy of a Flume Flow](https://github.com/mbkeane/BigDataTechCon/wiki/The-basic-anatomy-of-a-Flume-Agent-and-a-Flume-Flow)

[Exercise 1 Flume agent spooling from the file system using Kafka Channel and Kafka Sink](https://github.com/mbkeane/BigDataTechCon/wiki/Exercise-1)

[Exercise 2 Add a second flume agent with a Kafka source and chained interceptors](https://github.com/mbkeane/BigDataTechCon/wiki/Exercise-2)

[Exercise 3 Use Camus to pull data from Kafka to HDFS](https://github.com/mbkeane/BigDataTechCon/wiki/Exercise-3)

[Exercise 4 Debug and Camu map reduce job](https://github.com/mbkeane/BigDataTechCon/wiki/Exercise-4)


### The Motivation Behind this Tutorial
Part of my teams responsiblity at [Conversant](https://conversantmedia.com) is maintain Flume flows which collect 100 billion log lines per day.  In the spring of 2015 our current Flume agents were writing to (or 'sinking' in Flume parlance) a Hadoop cluster using the  [MapR](https://mapr.com) distribution.  A new [Cloudera](https://conversantmedia.comhttp://www.cloudera.com/content/www/en-us.html) hadoop cluster had been stood up and I was tasked with the "trivial" task of migrating the Flume flow end point from the MapR cluster to the Cloudera cluster.  Beyond the distributions differenced the work load difference was vastly different. The MapR cluster was owned by my team.  All jobs run on this cluster were map reduce jobs owned by my team.  The Cloudera cluster was shared by multiple teams including an army of decision scientists learning how to write [Spark](http://spark.apache.org) jobs.  My trivial task became my spring 2015 nightmare.  I had heard stories of Flume not playing well with Cloudera and [Hortonworks](http://hortonworks.com) hadoop distributions.  Rarely had we seen a problem on the MapR cluster.  Yes the Flume agents sinking to the MapR cluster would backup when the cluster was under heavy load, particulary during the "copy" phase of large mapreduce jobs but ultimately the copy phase completed and the Flume backup drained with just a delay in data arrival.  On the new Cloudera cluster the results were not the same.  Resource demands were much higher.  Flume has no YARN integration and would consistently fail to open, close or rename files when writing to the cluster.  Thousands of timeout exceptions were written to the namenode and datanode logs.  Untimately there was data loss and data corruption.

The other half of the data team at [Conversant](https://conversantmedia.com) was working with [Kafka](http://kafka.apache.org) and [Storm](http://storm.apache.org) and having great results.  [Flafka](http://blog.cloudera.com/blog/2014/11/flafka-apache-flume-meets-apache-kafka-for-event-processing) or Flume integration with Kafka was well underway and scheduled to be included in the next [Flume](http://flume.apache.org) release, 1.6.  Fluming into Kafka would decouple Flume from the Cloudera cluster and colleagues of mine had heard good things about [LinkedIn's](https://www.linkedin.com) open source Kafka to HDFS pipeline, [Camus](https://github.com/linkedin/camus).  Camus is a map reduce framework runs map reduce jobs with Kafka as an source and HDFS as an destination.  As a map reduce job it could run in its own queue with resource management by YARN.  Could this solve our problem.  The answer was a resounding yes.  In less than 6 weeks we were live and today we are "Camusing" 100 billion log lines from Kafka to HDFS daily.

Based on the success we had integrating Flume, Kafka and Camus into our log processing a colleague of mine suggested I put together tutorial for [BigDataTechCon](http://www.bigdatatechcon.com/).  This is the output of that effort.


The integration of Flume + Kafka + Camus does require implementing a couple Camus interfaces.  At Conversant we have created quite a few Flume pluggins as well.  My goal with this tutorial is to get users up and running with Flume, Kafka and Camus in a development environment as quickly as possible.  While implementing two Camus interfaces is all it would take to demonstrate a full data collection pipeline I have included two maven projects in this repo, "flume" and "camus".  An interesting "Flafka" caveat prompted me to also make a simple flume pluggin (Interceptor interface implementation).  More on this later in the tutorial. Also, I wanted to get users up and running in an environment where they could step through their code in a debuggger.  Kafka is used straight out of the box.  Flume will from within your IDE.  Camus jobs being map reduce job must be submitted to hadoop, The Camus job will run in local mode with a demonstration of running in local mode with remote debugging enabled.



**NOTE** this tutorial is not meant to be an indepth study of Flume, Kafka or Camus.  For a better understanding of Flume and its use cases I recommend Steve Hoffman's [Apache Flume: Distributed Log Collection for Hadoop - Second Edition](http://www.amazon.com/Apache-Flume-Distributed-Collection-Hadoop/dp/1784392170/ref=sr_1_2?s=books&ie=UTF8&qid=1446338183&sr=1-2&keywords=flume).
