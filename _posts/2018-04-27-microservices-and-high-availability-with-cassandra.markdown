---
layout: post
title: Smart Meters, Micro-Services and High Availability with Cassandra
date: '2018-04-27 11:56:34'
categories: 'Tech'
author: Satish Ramjee
tags:
- microservices
- smart-meters
- cassandra
---

## Smart Meter Data

In the energy sector, smart meter readings are received on a daily basis. These readings can have upto half-hourly interval readings for the previous day (or more). Consumer Access Devices (CAD) can potentially send meter reads every few mins. With a large portfolio (> 1M) of smart meters this can add up to quite a sizeable amount of data being gathered daily. 

Meter readings are very well suited to be captured as a history of events ([time series](https://en.wikipedia.org/wiki/Time_series)). Every half an hour, day or month a reading is taken from the meter and sent to the supplier. 

The data can then be packed as a unit (event stream set) and aggregated over, to gather usage data for billing, reporting and for settlement with industry. The read events themselves can provide customer analytics to display energy consumption behavior. 

![Screen-Shot-2018-04-27-at-00.35.50](/content/images/2018/04/Screen-Shot-2018-04-27-at-00.35.50.png)

## Highly Available
Cassandra (C*) promises always on availability, linear scalability and speed. This is provided that the number of cassandra nodes, consistency level (CL), replication factor (RF) etc. are set up and configured correctly.  See [Building HA C*](https://www.slideshare.net/rastrick/always-on-building-highly-available-applications-on-cassandra). For sizing, CL and RF, see this [calculator](https://www.ecyrd.com/cassandracalculator/).

An availability failure test is a very worthwhile excercise. Turn off one of your nodes and see what happens. Hopefully all is still well and your CL in your code is correctly set for your needs. 

Adding extra nodes as the number of customers increases is seamless with no downtime. C* employs a masterless data replication mechanism using the [gossip](https://www.edureka.co/blog/gossip-protocol-in-cassandra/) protocol.

Within the [CAP](https://en.wikipedia.org/wiki/CAP_theorem)  theory, C* is more aligned to A & P but believe it or not it can also provide full consistency although at the loss of availability (with the CL set to ALL). 

Eventually consistency with CL set to ANY for writes and reads is good enough for our business case, where we employ [BASE](http://www.dataversity.net/acid-vs-base-the-shifting-ph-of-database-transaction-processing/) instead of ACID guarantees.

## Event Sourcing
Cassandra is an ideal persistence technology for storing events, and the data structures needed for meter reads are not too onerous. Basically, you have the reading, time stamp, optionally the meter type (gas, electric), the register (economy 7 for electricity). This can be saved in a manner that can be sorted by some criteria such as day. The querying and sorting criteria has to be determined in advance when creating the keyspace table structure. Querying and sorting is then based on the structure defined. That is, the partition key and clustering key (in the case of a compound key).  

The partition key defines which node the set of data lives on, while the clustering key determines the sorting order in the node. There is also a concept of composite partition keys where the partition can be further split into which nodes they sit in. A lot more on this at [DataStax](https://www.datastax.com/dev/blog/the-most-important-thing-to-know-in-cassandra-data-modeling-the-primary-key).

### Queries and Reporting
There are no joins in C* and query is limited to how the data structure is defined. Spark can be used to query the cassandra node designated for this but a better way might be to pump the reads to hadoop via kafka and use tools such as impala, in order to reduce load on the C* nodes. Another option is to use [Apache Ignite](https://ignite.apache.org) and back that with C*, which also provides ACID guarantess, as described [here](https://www.infoworld.com/article/3191895/application-development/light-a-fire-under-cassandra-with-apache-ignite.html). 

[Spark streaming](https://academy.datastax.com/resources/apache-spark-streaming) is another alternative but requires a carefully tuned cluster, and is more appropriate for real time analysis.

### Idempotency (Replay and Self Healing)
As data is received, a common practice is to transform (say from files) and push the data into message queues. Services then work the events from the queues and store results into C*. Idempotency is helpful here, the data structure needs to be designed accordingly (upsert) so that the files can be safely replayed without creating new entries (insert). C* is well suited to this, as long as a new timestamp is not created and the time is derived from the original data. See [C* Data structure design](https://teddyma.gitbooks.io/learncassandra/content/client/read_requests.html) for more information. Using timestamp itself doesn't guarantee uniqueness across the cluster, C* provides TimeUUID for this.

### Microservices
A vertical layer of services that fulfill a business function is the way to go. Avoid the temptation of architecting 'common' horizontal services that create dependencies that then become blockers when managing multiple delivery teams. 

As well as be read from, smart meters can also be written to, to set customer preferences such as read frequency, vending parameters (for payment meters),  tariff details, turning off reading of meters etc. 

Many of the above fulfill a defined bounded context which form an ideal microservice business domain and vertical concern. 

Being able to fully test your microservice in a container (docker) locally before deployment to an environment is a big plus. This gives rapid feedback and the ability to fix things quickly. 

Using testing for development, listening and working with them is a practice that takes a change of mindset but reaps great benefits in quality and responsiveness. The tests not only drive incremental product delivery and lean code but also enables defect prevention (as opposed to defect detection).

In addition, a good DevOps practice is needed to help with managing the services (whether on cloud or on premise). This involves service discovery (eg. [consul](https://www.consul.io), [registrator](https://hub.docker.com/r/gliderlabs/registrator/)), [monitoring/dashboards](https://github.com/Smashing/smashing), visualisation [vizceral](https://github.com/Netflix/vizceral), reporting (hadoop/HDFS, impala), traffic tracing and fault finding ([newrelic](https://newrelic.com)).

Of course there is a lot out there about microservices, devops and testing....

Sam Newman's book, [Building Microservices](https://www.amazon.co.uk/Building-Microservices-Sam-Newman/dp/1491950358/ref=sr_1_1?ie=UTF8&qid=1524785065&sr=8-1&keywords=building+microservices), is a great help. 

#### Large Data
The data received from meters can grow quite large, especially if the meter hasn't communicated for a while. The whole set should not be pushed onto the queue rather just a reference for the services to read the data from. Otherwise you run the risk of clogging up both the queue and using up the memory in the microservice space. AWS S3 is also useful here to keep a backup.

If the data is file based, this allows streaming mechanisms to read 'streams' and reduce memory usage. 

Integration software such as Apache Camel or Spring Integration provides many tools to achieve this.

#### Distributed Locking
If multiple instances of a service use sFTP (eg within Camel), there will be a need to ensure no two instances pick up the same file. The downstream services ought to be stateless and should not have to keep a track of what has been processed.

Although there is a lock feature in Camel, this is not guaranteed to function across different server nodes.

A simple but crude option is to provide this through a traditional relational database, with the unique filename being a primary key and a successful insert allowing the service to proceed. This has its own limitations as you can imagine.  

NB: Spring Cloud provides a global lock feature which is worth looking into. 



## Big Data Analytics Presentation

For more on this, you can [view](https://www.youtube.com/watch?v=e_iSe6TyXD8&t=11s) this presentation recorded at the BDA conference June 2017.

# Disclaimer
Although much of the observations here are based on experiences while working for an energy company, [First Utility](https://www.first-utility.com/isave-fixed?promo=BINGBRAND&utm_campaign=New%20-%20Brand%20-%20Keyword%20-%20Exact&utm_medium=cpc&utm_source=bing&utm_term=first%20utility), the concepts described here are general to any large data processing problem with microservices and does not directly relate to any specific company process or practice.





