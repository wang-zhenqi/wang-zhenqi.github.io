---
layout: article
title: "AWS Data Analytics Specialty (DAS-C01) 认证例题整理（一）"
description: "这篇文章里包含了笔者整理的关于 DAS-C01 认证的样题，共 50 道题。这些题目都是从网站、论坛上搜集来的。答案是笔者通过查阅资料并结合他人解题思路后得出的，不能保证正确性。读者需认真思考，仔细辨别，欢迎提出不同意见。"
tags: [aws, das-c01, reference]
---

## Q001

`#kinesis-data-streams` `#kinesis-data-firehose` `#redshift` `#athena` `#collection` `#processing`

A financial services company needs to aggregate daily stock trade data from the exchanges into a data store. The company requires that data be `streamed directly into the data store`, but also `occasionally allows data to be modified using SQL`. The solution should `integrate complex, analytic queries` running `with minimal latency`. The solution must `provide a business intelligence dashboard` that enables viewing of the top contributors to anomalies in stock prices.  
Which solution meets the company's requirements?

A. Use Amazon Kinesis Data Firehose to stream data to Amazon S3. Use Amazon Athena as a data source for Amazon QuickSight to create a business intelligence dashboard.

B. Use Amazon Kinesis Data Streams to stream data to Amazon Redshift. Use Amazon Redshift as a data source for Amazon QuickSight to create a business intelligence dashboard.

C. Use Amazon Kinesis Data Firehose to stream data to Amazon Redshift. Use Amazon Redshift as a data source for Amazon QuickSight to create a business intelligence dashboard.

D. Use Amazon Kinesis Data Streams to stream data to Amazon S3. Use Amazon Athena as a data source for Amazon QuickSight to create a business intelligence dashboard.

### Answer - C

这道题的重点是关于 _SQL_ 和 _queries_ 的描述：“可以用 SQL 来修改数据，能够以最小的延迟集成复杂的分析查询，同时还能做 data store”。能够实现这些的就只有 **Redshift**. 而流服务中可以将数据传入 Redshift 的就是 **Kinesis Data Firehose**. 至于 BI 仪表盘自然就是 **QuickSight**了。

按排除法来做的话，_Athena_ 不能通过 SQL 修改数据，最多将查询结果插入到另一个指定的表中，因此排除带有 Athena 的选项 A、D；Kinesis Data Streams 不能将数据传入 Redshift，因此排除 B.

## Q002

`#quicksight` `#security`

A financial company hosts a data lake in Amazon S3 and a data warehouse on an Amazon Redshift cluster. The company uses Amazon QuickSight to build dashboards and wants to `secure access from its on-premises Active Directory to Amazon QuickSight`.  
How should the data be secured?

A. Use an Active Directory connector and single sign-on (SSO) in a corporate network environment.

B. Use a VPC endpoint to connect to Amazon S3 from Amazon QuickSight and an IAM role to authenticate Amazon Redshift.

C. Establish a secure connection by creating an S3 endpoint to connect Amazon QuickSight and a VPC endpoint to connect to Amazon Redshift.

D. Place Amazon QuickSight and Amazon Redshift in the security group and use an Amazon S3 endpoint to connect Amazon QuickSight to Amazon S3.

### Answer - A

要使从本地的 _Active Directory_ 到 _QuickSight_ 的访问是安全的，就需要使用 QuickSight 企业版的 **AD Connector** 进行连接，同时使用 SSO 进行认证。

## Q003

`#emr` `#availability` `#cost-effective`

A real estate company has a mission-critical application using Apache HBase in Amazon EMR. Amazon EMR is configured with a single master node. The company has over 5 TB of data stored on an Hadoop Distributed File System (HDFS). The company wants a `cost-effective` solution to make its HBase data `highly available`.
Which architectural pattern meets company's requirements?

A. Use Spot Instances for core and task nodes and a Reserved Instance for the EMR master node. Configure the EMR cluster with multiple master nodes. Schedule automated snapshots using Amazon EventBridge.

B. Store the data on an EMR File System (EMRFS) instead of HDFS. Enable EMRFS consistent view. Create an EMR HBase cluster with multiple master nodes. Point the HBase root directory to an Amazon S3 bucket.

C. Store the data on an EMR File System (EMRFS) instead of HDFS and enable EMRFS consistent view. Run two separate EMR clusters in two different Availability Zones. Point both clusters to the same HBase root directory in the same Amazon S3 bucket.

D. Store the data on an EMR File System (EMRFS) instead of HDFS and enable EMRFS consistent view. Create a primary EMR HBase cluster with multiple master nodes. Create a secondary EMR HBase read-replica cluster in a separate Availability Zone. Point both clusters to the same HBase root directory in the same Amazon S3 bucket.

### Answer - B

A 选项：Spot Instance 不适合作为 core node，尽管它很便宜，但是它随时可能丢失，无法保证 core node 上数据的持久性。（但 spot instance 可以作为 task node）

C 选项：EMR 不支持多个集群上的 HBase 的根目录指向同一个 S3 bucket。（参考：[HBase on Amazon S3 (Amazon S3 storage mode) - Enabling HBase on Amazon S3](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hbase-s3.html#emr-hbase-s3-enable)）

B 和 D 选项是争议比较多的。首先这两种方案都是可实现的，它们之间的差别就在于 B 选项只是创建了一个多主节点的集群，而 D 选项还多加了一个只读的副集群。加一个 read-replica cluster 的好处在于这个集群可以创建在另一个可用区（Availability Zone）上，这样当主集群不可用时，副集群还可以正常进行读操作。而最终我更倾向于 B 选项的原因是，题中主要强调的是 cost-effective 和 data highly available，D 选项的缺点就在于成本会更大，而且强化的是集群的可用性；而对于数据来讲，在 S3 上存储的 EMRFS 已经可以使数据可用性足够高了。

另外：题中提到的 EMRFS consistent view 主要是为了提高数据访问的一致性，利用 DynamoDB 存储元数据来追踪 EMRFS 上的数据，这样还会产生额外的 DynamoDB 的费用。由于 S3 自 2020-12-01 起添加了 strongly consistency 的特性，因此现在已经不再需要 EMRFS consistent view 了，从 2023-06-01 开始，新的 EMR 版本将不再将其作为配置选项，这样还能节约成本。（参考：[EMR - Consistent view](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-plan-consistent-view.html)）

## Q004

`#kinesis-data-streams` `#kinesis-data-firehose` `#quicksight` `#opensearch` `#log-analysis`

A software company hosts an application on AWS, and new features are released weekly. As part of the application testing process, a solution must be developed that `analyzes logs` from each Amazon EC2 instance to ensure that the application is working as expected after each deployment. The collection and analysis solution should be `highly available` with the ability to display new information with `minimal delays`.  
Which method should the company use to collect and analyze the logs?

A. Enable detailed monitoring on Amazon EC2, use Amazon CloudWatch agent to store logs in Amazon S3, and use Amazon Athena for fast, interactive log analytics.

B. Use the Amazon Kinesis Producer Library (KPL) agent on Amazon EC2 to collect and send data to Kinesis Data Streams to further push the data to Amazon OpenSearch Service (Amazon Elasticsearch Service) and visualize using Amazon QuickSight.

C. Use the Amazon Kinesis Producer Library (KPL) agent on Amazon EC2 to collect and send data to Kinesis Data Firehose to further push the data to Amazon OpenSearch Service (Amazon Elasticsearch Service) and OpenSearch Dashboards (Kibana).

D. Use Amazon CloudWatch subscriptions to get access to a real-time feed of logs and have the logs delivered to Amazon Kinesis Data Streams to further push the data to Amazon OpenSearch Service (Amazon Elasticsearch Service) and OpenSearch Dashboards (Kibana).

### Answer - C

首先排除 A 选项，detailed monitoring 监测的是 EC2 实例的运行状况，和 basic monitoring 的差别在于它可以每分钟发布一次监测数据，同时要收费。它并不能监测运行在 EC2 上的应用程序的日志文件。另外，利用 Athena 来做实时的日志分析也是不合理的，因为 Athena 原本只是为了交互式的数据分析设计的，而题中要求的是“display new information with minimal delays”。

B、C 选项中用词不够准确，争议也源自于此。KPL agent 不是在 Kinesis 的定义里并不存在，但是却有一个 Kinesis agent 的应用程序，它是基于 JAVA 的独立的程序，可以将消息传递给 Kinesis Data Stream 或者 Kinesis Firehose. 如果理解为 Kinesis agent，那么 C 选项就合理了。B 选项中的错误在于 QuickSight 不能可视化 OpenSearch 中的内容，QuickSight 主要的数据源是 Redshift、Athena、Aurora、文件，以及各种搭建在 EC2 上的数据库，总得来说就是各种数据库系统。OpenSearch 的结果可视化是用其内建的 Kibana 完成的。

D 选项中，CloudWatch Subscription 确实可以实时地获取应用程序日志中的信息，可以自定义 metrics 来进行监测，也可以将监测结果传到 Kinesis Data Streams。但是 KDS 的数据不能直接进入 OpenSearch，只有 Kinesis Firehose 才可以。
因此这道题的答案我更倾向于 C 选项。

## Q005

`#glue` `#optimization`

A data analyst is using AWS Glue to organize, cleanse, validate, and format a `200 GB dataset`. The data analyst triggered the job to run with the Standard worker type. After 3 hours, the AWS Glue job status is still RUNNING. Logs from the job run show `no error codes`. The data analyst wants to improve the job execution time `without over-provisioning`.  
Which actions should the data analyst take?

A. Enable job bookmarks in AWS Glue to estimate the number of data processing units (DPUs). Based on the profiled metrics, increase the value of the executor- cores job parameter.

B. Enable job metrics in AWS Glue to estimate the number of data processing units (DPUs). Based on the profiled metrics, increase the value of the maximum capacity job parameter.

C. Enable job metrics in AWS Glue to estimate the number of data processing units (DPUs). Based on the profiled metrics, increase the value of the spark.yarn.executor.memoryOverhead job parameter.

D. Enable job bookmarks in AWS Glue to estimate the number of data processing units (DPUs). Based on the profiled metrics, increase the value of the num- executors job parameter.

### Answer - B

首先区分一下 job bookmarks 和 job metrics：前者是对已处理的数据做的记录，相当于是 checkpoint，这样在任务重复执行的时候就可以从记录点开始，而不用从头处理；job metrics 是对任务运行状况的监测，例如 CPU、内存使用率，哪个 excecutor 一直被占用，运行时长，读写量等等。由此可以看出，针对题中的问题，应该利用 job metrics 来判断所需 DPU 的数量。由此排除选项 A、D。

选项 B、C 中提到的两个参数，`spark.yarn.executor.memoryOverhead` 指的是 executor 所需的堆外内存的大小。题目里描述到没有报错，说明不是 OOM 的原因，只是因为计算资源不够才导致任务迟迟不能完成，因此应该增加集群的“最大容量”以增加计算资源。

## Q006

`#glue` `#redshift` `#loading-data`

A company has a business unit uploading .csv files to an Amazon S3 bucket. The company's data platform team has set up an AWS Glue crawler to do discovery, and create tables and schemas. An AWS Glue job writes processed data from the created tables `to an Amazon Redshift database`. The AWS Glue job handles column mapping and creating the Amazon Redshift table appropriately. When the AWS `Glue job is rerun` for any reason in a day, `duplicate records` are introduced into the Amazon Redshift table.  
Which solution will update the Redshift table without duplicates when jobs are rerun?

A. Modify the AWS Glue job to copy the rows into a staging table. Add SQL commands to replace the existing rows in the main table as postactions in the DynamicFrameWriter class.

B. Load the previously inserted data into a MySQL database in the AWS Glue job. Perform an `upsert` operation in MySQL, and copy the results to the Amazon Redshift table.

C. Use Apache Spark's DataFrame dropDuplicates() API to eliminate duplicates and then write the data to Amazon Redshift.

D. Use the AWS Glue ResolveChoice built-in transform to select the most recent value of the column.

### Answer - A

首先要说明的是题目中的问题主要是由于 Redshift 不支持 upsert 操作所导致的。解决方案就是用某种方式在更新 Redshift 表时去掉重复记录。

D 选项是可以最先排除的，因为 ResolveChoice 是针对同一 DynamicFrame 中拥有多个数据类型的字段的。例如：`"MyList": [{"price": 100.00}, {"price": "$100.00"}]`. 通过 ResolveChoice 可以将 `price` 字段的类型统一，或者分成两个带后缀的新字段：`price_int`, `price_string`。

C 选项中提到的 Spark DataFrame 的 dropDuplicates 方法，它是针对同一个 DataFrame 进行的去重。如果要在 Redshift 表中使用该方法来去重，那必然需要将整个表都读入一个 DataFrame，这种方案很显然会非常费事，也很浪费资源。

B 选项也是一个可行的方案，但并非最优解。原因和 C 选项一样，也需要先将 Redshift 表中数据加载到 MySQL 中，upsert 之后再把数据导回 Redshift。这里除了会在数据传输上浪费大量资源之外，还涉及到 MySQL 的性能问题，以及如何将数据再复制回 Redshift 的问题。要想将数据复制回 Redshift 就需要 truncate 原先的表，再用 load 命令加载数据。

A 选项是最高效的。首先 Glue 处理的数据只需往 Redshift 上写一次，写入一个临时表。再从目标表中删除临时表中重复的数据，可以用 `DELETE FROM {target_table} USING {staging_table} WHERE {condition}` 的语句。最后直接运行 `INSERT INTO {target_table} SELECT * FROM {staging_table}` 即可。参考文档：[https://aws.amazon.com/premiumsupport/knowledge-center/sql-commands-redshift-glue-job/](https://aws.amazon.com/premiumsupport/knowledge-center/sql-commands-redshift-glue-job/), [https://docs.aws.amazon.com/redshift/latest/dg/merge-examples.html](https://docs.aws.amazon.com/redshift/latest/dg/merge-examples.html)

## Q007

`#kinesis-data-streams` `#athena` `#optimization`

A streaming application is reading data from Amazon Kinesis Data Streams and immediately writing the data to an Amazon S3 bucket `every 10 seconds`. The application is reading data from `hundreds of shards`. The batch interval cannot be changed due to a separate requirement. The data is being accessed by Amazon  
Athena. Users are seeing `degradation in query` performance `as time progresses`.  
Which action can help improve query performance?

A. Merge the files in Amazon S3 to form larger files.

B. Increase the number of shards in Kinesis Data Streams.

C. Add more memory and CPU capacity to the streaming application.

D. Write the files to multiple S3 buckets.

### Answer - A

题目中描述道，Athena 的查询性能是随着时间增加而下降的。这就说明导致性能下降的原因是在程序运行的过程中累积起来的。如果是因为 shard、CPU、memory 等计算、存储资源不够，那么在一开始性能就会不好。

从另一方面来看，应用程序每 10 秒向 S3 写入一个文件，按照 Kinesis Data Stream 的性能，这个文件最多就是 20M，那么上百个 shards 就会在 S3 上生成上百个不超过 20M 的小文件。Athena 在查询时需要不断地读取文件及其 metadata，而 S3 每秒只支持最多 5500 次 GET / HEAD 请求，小文件过多就会造成读取速度下降，影响性能。

因此解决方案就是将小文件合并成大文件，减少文件数量，降低查询请求次数。故而选 A。参考文档：[Performance tuning in Athena](https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html#performance-tuning-data-size)

## Q008

`#opensearch` `#optimization`

A company uses Amazon OpenSearch Service (Amazon Elasticsearch Service) to store and analyze its website clickstream data. The company `ingests 1 TB of data daily` using Amazon Kinesis Data Firehose and stores one day's worth of data in an Amazon ES cluster.  
The company has very slow query performance on the Amazon ES index and occasionally sees `errors from Kinesis Data Firehose` when attempting to write to the index. The Amazon ES cluster has `10 nodes running a single index` and 3 dedicated master nodes. Each data node has 1.5 TB of Amazon EBS storage attached and the `cluster is configured with 1,000 shards`. Occasionally, `JVMMemoryPressure errors` are found in the cluster logs.  
Which solution will improve the performance of Amazon ES?

A. Increase the memory of the Amazon ES master nodes.

B. Decrease the number of Amazon ES data nodes.

C. Decrease the number of Amazon ES shards for the index.

D. Increase the number of Amazon ES shards for the index.

### Answer - C

这道题纯粹是在考 OpenSearch 中 nodes、shards、index 之间的关系，以及如何按照数据量分配资源。

OpenSearch 的 master nodes 主要是用来管理 domain 的，并不进行数据的存储和分析。一般为了实现高可用，可以设置 3 个 master nodes，正如题目中描述的那样。

数据是在 data nodes 中存储和处理的。每一个 index 都是同一类型、同一领域的文件的集合。在 OpenSearch 中，index 被分割为多个 shards，分布在多个 data nodes 中。为了保证数据的可访问性，还会为每个 shard 都生成若干冗余 shard。也就是说，OpenSearch 中的数据，是有多个副本的、分散在了多个 data nodes 中的。另外，每一个 shard 都相当于一个 Lucene engine（[Apache Lucene](https://lucene.apache.org/), OpenSearch 背后的技术），自己就是一个搜索引擎，因此还会占用计算资源。

Shards 太大或者数量太多都会造成性能问题。如果 shards 太大，那么在错误恢复的时候就会花费过多的时间；如果数量太多，则会因为 shards 的运算而消耗掉 data nodes 上的资源。这也就是题目中“JVMMemoryPressure errors”的来源。题中描述的“时常会遇到 Kinesis Firehose 写入时发生错误”也进一步佐证了这个原因。所以解决方案是减少 index 划分出的 shards 的数量。

关于 shards 数量的选择，官方文档里提供了一个计算公式：`(source data + room to grow) * (1 + indexing overhead (about 10%)) / desired shard size = approximate number of primary shards`。所谓的 “desired shard size” 一般分两种情况：

1. 要求检索速度的，shard 的大小在 10 ~ 30 GB 之间
2. 写操作密集的，例如日志分析，大小在 30 ~ 50 GB 之间。

那么代入题目中的情景，单一 index，写操作密集，每天固定 1000 GB 的数据（故而 room to grow 可以看作 0），大概需要 `(1000 + 0) * (1 + 0.1) / 50 = 22` 个 shards，题目中用了 1000 个，显然太多了。

## Q009

`#redshift` `#s3` `#architecture`

A manufacturing company has been collecting IoT sensor data from devices on its factory floor for a year and is storing the data in Amazon Redshift for daily analysis. A data analyst has determined that, at an expected ingestion rate of about 2 TB per day, the cluster will be undersized in less than 4 months. A long-term solution is needed. The data analyst has indicated that most queries only reference the `most recent 13 months` of data, yet there are also quarterly reports that need to query all the data generated from the `past 7 years`. The chief technology officer (CTO) is concerned about the `costs, administrative effort, and performance` of a long-term solution.  
Which solution should the data analyst use to meet these requirements?

A. Create a daily job in AWS Glue to UNLOAD records older than 13 months to Amazon S3 and delete those records from Amazon Redshift. Create an external table in Amazon Redshift to point to the S3 location. Use Amazon Redshift Spectrum to join to data that is older than 13 months.

B. Take a snapshot of the Amazon Redshift cluster. Restore the cluster to a new cluster using dense storage nodes with additional storage capacity.

C. Execute a CREATE TABLE AS SELECT (CTAS) statement to move records that are older than 13 months to quarterly partitioned data in Amazon Redshift Spectrum backed by Amazon S3.

D. Unload all the tables in Amazon Redshift to an Amazon S3 bucket using S3 Intelligent-Tiering. Use AWS Glue to crawl the S3 bucket location to create external tables in an AWS Glue Data Catalog. Create an Amazon EMR cluster using Auto Scaling for any daily analytics needs, and use Amazon Athena for the quarterly reports, with both using the same AWS Glue Data Catalog.

### Answer - A

这里首先要注意的是两个时间长度：大多数查询需要用到最近 13 个月的数据，每季度的报告需要查询到过去 7 年的数据。前者说明近 13 个月的数据是频繁查询的，需要能够快速访问到；后者说明过去 7 年的数据都需要保留，但因为是每季度查询，因此访问速度可以慢一些。题目中说到集群的空间将在 4 个月内就不够用了，也就是说集群的空间大概在 `2TB * 120 = 240TB` 左右。从 Redshift 集群机器类型的配置可知，即使是 8XL 的 dense storage 机器，也需要 `240TB / 16TB = 15` 个计算节点。可以预见，如果继续将数据放在 Redshift 上，会产生高昂的费用。因此 B 选项（创建快照，使用 dense storage）是不可行的。

如果想要保留如此多的数据量，不影响 Redshift 的查询，同时还要降低数据存储的成本，那么就需要将数据转移到 S3 上，然后利用 Redshift Spectrum 将 S3 上的数据看作一张张结构化的数据表，与 Redshift 集群上的表联合使用。题目中提供了 3 种方案。

选项 A 和 C 均提出要将近 13 个月之前的数据移动到 S3 中，区别在于选项 A 的方式删除了 Redshift 上的数据，而选项 C 中仍然保留。很明显选项 A 更有利于优化存储成本，它只需要 Redshift 保留近 13 个月的数据即可。而选项 C 所需的 Redshift 的存储空间依然在不断上涨。

选项 D 和选项 A 的区别在于，选项 D 舍弃了 Redshift，把所有的数据都放在了 S3 上，利用 EMR 来进行数据分析，再用 Athena 来查询结果。这样依赖增加了管理的难度，没有利用到 Redshift 数据仓库的特性。不符合要求。

## Q010

`#glue`

An insurance company has raw data in JSON format that is sent `without a predefined schedule` through an Amazon Kinesis Data Firehose delivery stream to an Amazon S3 bucket. An AWS Glue crawler is scheduled to run `every 8 hours` to update the schema in the data catalog of the tables stored in the S3 bucket. Data analysts analyze the data using Apache Spark SQL on Amazon EMR set up with AWS Glue Data Catalog as the metastore. Data analysts say that, `occasionally, the data they receive is stale`. A data engineer needs to provide access to the most up-to-date data.
Which solution meets these requirements?

A. Create an external schema based on the AWS Glue Data Catalog on the existing Amazon Redshift cluster to query new data in Amazon S3 with Amazon Redshift Spectrum.

B. Use Amazon CloudWatch Events with the rate (1 hour) expression to execute the AWS Glue crawler every hour.

C. Using the AWS CLI, modify the execution schedule of the AWS Glue crawler from 8 hours to 1 minute.

D. Run the AWS Glue crawler from an AWS Lambda function triggered by an S3:ObjectCreated:\* event notification on the S3 bucket.

### Answer - D

从题目描述中可以看出，导致数据过期的原因是 Glue crawler 的运行间隔太长，导致有时拥有新的 schema 的数据接入后，schema 没能及时更新。如果采用缩短 crawler 的运行间隔的方式，只要运行间隔和接入周期是匹配的，那么对于周期性接入数据的情景就是有效的。

然而题目中的场景是接入数据是随机的，因此 crawler 就需要由事件触发运行。选项 B、C 都仅仅减小了运行间隔，仍然有可能出现 schema 更新不及时的情况。选项 A 的方案对于解决问题没有帮助，因为 Redshift Spectrum 同样依赖于 Glue data catalog 里记录的 schema，一样会遇到更新不及时的问题。因此答案为 D。

## Q011

`#S3` `#optimization`

A company that produces network devices has millions of users. Data is collected from the devices on `an hourly basis` and stored in an Amazon S3 data lake.  
The company runs analyses on the `last 24 hours` of data flow logs for abnormality detection and to troubleshoot and resolve user issues. The company also analyzes historical logs dating `back 2 years` to discover patterns and look for improvement opportunities.  
The data flow logs contain many metrics, such as date, timestamp, source IP, and target IP. There are about `10 billion events every day`.
How should this data be stored for optimal performance?

A. In Apache ORC partitioned by date and sorted by source IP

B. In compressed .csv partitioned by date and sorted by source IP

C. In Apache Parquet partitioned by source IP and sorted by date

D. In compressed nested JSON partitioned by source IP and sorted by date

### Answer - A

这道题主要是在对比数据文件的类型以及分区排序的不同对于查询性能的影响。首先可以在题目限定之外对比一下不同选择的优劣。

从文件类型上来讲，主要分为结构化的行式存储（csv）、列式存储（orc、parquet）以及半结构化（json）的形式。

1. 结构化存储的优点是，schema 固定，字段数量、类型、限制都是确定的，便于程序读取和处理。
   1. 行式存储的好处在于，每一条数据就对应一行，数据以行为单位记录在文件中，便于处理，数据也很直观。这种便利性也会带来占用空间的冗余性，即使某一些字段上没有值，也依然需要将它们罗列出来。
   2. 列式存储则是将同一列的数据聚集在一起，而同一列的数据都是属于同一个数据类型（格式），因此便于压缩。此外遇到没有数据的单元格，列式存储可以直接略过这一字段，进一步缩减了空间占用。
2. 半结构化存储的优缺点与结构化存储正好互补：可以省略没有值的单元格，可以不定义数据类型，占用空间少，但是处理起来需要花费更多的操作。

从分区排序的角度来说，数据以文件的形式存储在 S3 上，如果能对这些文件恰当地分区（按某些字段划分，将数据文件存储在子文件夹中）、排序（按某些字段进行排序），那么会大大减少扫描的数据量，从而加快查询速度。

回到题目中来，按其描述，每天会有 100 亿条数据，存储如此多的数据需要更好的压缩水平，因此行式存储是优于列式存储的和半结构化存储的，排除选项 B、D。另外，数据按小时传入 S3 bucket 并进行分析，还需分析 2 年前的数据，这说明按时间分区会使查询最为容易（最快定位，最少扫描）。如果按照 source IP 分区，那么对于同一天的数据，有多少个 source IP，一次查询就要扫描多少个分区，速度会很慢。而按照 source IP 分区会有利于数据过滤以及连接操作，因此选 A。

## Q012

`#security` `#redshift` `#hardware-security-module`

A banking company is currently using an Amazon Redshift cluster with dense storage (DS) nodes to store sensitive data. An audit found that the cluster is unencrypted. Compliance requirements state that a database with `sensitive data must be encrypted through a hardware security module` (HSM) with automated key rotation.  
Which combination of steps is required to achieve compliance? (Choose two.)

A. Set up a trusted connection with HSM using a client and server certificate with automatic key rotation.

B. Modify the cluster with an HSM encryption option and automatic key rotation.

C. Create a new HSM-encrypted Amazon Redshift cluster and migrate the data to the new cluster.

D. Enable HSM with key rotation through the AWS CLI.

E. Enable Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) encryption in the HSM.

### Answer - AC

这道题考察 Redshift 的 HSM 的应用。题中要求给现有的未加密的 cluster 添加 HSM 加密功能。首先，HSM 需要与 Redshift 建立可信连接，这就需要用户拥有证书，选项 A 正确；其次，HSM 只能在创建集群的时候设置，因此只能新建一个支持 HSM 的集群，然后把现有的集群迁移过来，选项 C 正确，排除选项 B、D。参考：[Amazon Redshift database encryption](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html)

选项 E 只是一个在非安全通信信道上进行密钥交换的机制，与给集群添加 HSM 功能没有关系。参考：[Elliptic-curve Diffie–Hellman](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman)

## Q013

`#glue` `#s3` `#dms`

A company is planning to do a proof of concept for a machine learning (ML) project using Amazon SageMaker with a subset of `existing on-premises data` hosted in the company's `3 TB data warehouse`. For part of the project, AWS `Direct Connect is established and tested`. To prepare the data for ML, data analysts are performing data curation. The data analysts want to `perform multiple step`, including mapping, dropping null fields, resolving choice, and splitting fields. The company needs the `fastest solution` to curate the data for this project.  
Which solution meets these requirements?

A. Ingest data into Amazon S3 using AWS DataSync and use Apache Spark scrips to curate the data in an Amazon EMR cluster. Store the curated data in Amazon S3 for ML processing.

B. Create custom ETL jobs on-premises to curate the data. Use AWS DMS to ingest data into Amazon S3 for ML processing.

C. Ingest data into Amazon S3 using AWS DMS. Use AWS Glue to perform data curation and store the data in Amazon S3 for ML processing.

D. Take a full backup of the data store and ship the backup files using AWS Snowball. Upload Snowball data into Amazon S3 and schedule data curation jobs using AWS Batch to prepare the data for ML.

### Answer - C

题目中的 SageMake、Direct Connect 其实和这道题的解答没有太大关系。题目中的场景要求的是如何最快速地对 3TB 的本地数据实行 ETL 操作。

依次来分析一下几个选项中提到的方案和服务：

1. AWS DataSync：主要是用于文件的迁移，可以在本地文件系统、其他服务商提供的文件系统与 AWS 上的多种文件系统之间进行迁移。常见的用例可以参考[这里](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html#use-cases)，可以看到并不适用于整个数据库或是数据仓库的迁移。
2. DMS：数据库迁移服务，可以从多种源数据库迁移到目标数据库，参考 **[AWS 数据库迁移服务简介]({{ site.baseurl }}{% link _posts/2023-04-22-introduction-to-aws-dms.md %})**。
3. AWS Glue：提供 data catalog、data integration and ETL 等服务，正如其名，它非常适合对数据进行编目、处理和集成，就像是胶水一样，将不同来源的数据黏合在一起。
4. Snowball：将本地数据加载到 S3 上的服务，适用于对大批量的数据进行加载。
5. AWS Batch：由 AWS 全托管的批处理服务，用户需自定义批处理的程序，使其运行在 AWS 的容器服务上，AWS Batch 会自动为任务分配资源，弹性伸缩。
6. Apache Spark：基于内存的数据处理引擎，以 RDD 或者 DataSet/DataFrame 为数据处理单位。AWS Glue 内置的处理引擎就是 Spark。

对于这道题目来说，首先没有必要在本地进行 ETL 的操作，排除选项 B。其次 AWS DataSync 不适用于数据仓库的迁移，另外编写 Spark 脚本也会付出额外的时间，从快速验证 POC 的角度来说并不经济，排除选项 A。最后，Snowball + Batch 的组合相当麻烦，申请 Snowball 的设备就需等待几天时间，再加上编写 Batch 的运行脚本的时间，整个过程耗时太久，另外 Batch 也不适合做大规模数据的 ETL，Spark 更适合处理这样的数据，排除选项 D。

再看最后的选项 C，数据仓库的迁移正好是 DMS 的使用场景之一，然后再用 Glue 的 ETL 控制台，可以快速地、可视化地搭建 ETL 任务，对于题目中提到的“mapping, dropping null fields, resolving choice, and splitting fields”等操作，Glue ETL 都提供了现成的模板。同时，Glue 内部也是使用 Spark 来处理数据的，这个选项兼顾了数据处理的速度与开发的速度，因此是最优解。

## Q014

`#quicksight` `#security` `#redshift` `#cross-region-access`

A US-based sneaker retail company launched its global website. All the transaction data is stored in Amazon RDS and curated historic transaction data is stored in Amazon Redshift in the `us-east-1 Region`. The business intelligence (BI) team wants to enhance the user experience by providing a dashboard for sneaker trends.  
The BI team decides to use Amazon QuickSight to render the website dashboards. During development, a team in Japan provisioned Amazon QuickSight in `ap-northeast-1`. The team is having difficulty connecting Amazon QuickSight from ap-northeast-1 to Amazon Redshift in us-east-1.  
Which solution will solve this issue and meet the requirements?

A. In the Amazon Redshift console, choose to configure cross-Region snapshots and set the destination Region as ap-northeast-1. Restore the Amazon Redshift Cluster from the snapshot and connect to Amazon QuickSight launched in ap-northeast-1.

B. Create a VPC endpoint from the Amazon QuickSight VPC to the Amazon Redshift VPC so Amazon QuickSight can access data from Amazon Redshift.

C. Create an Amazon Redshift endpoint connection string with Region information in the string and use this connection string in Amazon QuickSight to connect to Amazon Redshift.

D. Create a new security group for Amazon Redshift in us-east-1 with an inbound rule authorizing access from the appropriate IP address range for the Amazon QuickSight servers in ap-northeast-1.

### Answer - D

关于 QuickSight cross-region access，在 Udemy 的课程里专门有提到，使用 VPC 的方式无法解决 QuickSight 不能跨域访问 Redshift 的问题，解决方案是给 Redshift 添加一个安全组，将 QuickSight 的 IP 范围加入 inbound rule 中，因此选择 D。选项 A 中的“在 ap-northwest-1 区域创建 Redshift 的快照”太过复杂，而且需要额外支出，所以并不合理。参考：[https://docs.aws.amazon.com/quicksight/latest/user/enabling-access-redshift.html](https://docs.aws.amazon.com/quicksight/latest/user/enabling-access-redshift.html)

## Q015

`#redshift` `#redshift-spectrum` `#uncertain-answer`

An airline has .csv-formatted data stored in Amazon S3 with an AWS Glue Data Catalog. Data analysts want to join this data with call center data stored in  
Amazon Redshift as part of a daily batch process. The Amazon Redshift cluster is already under a heavy load. The solution must be `managed, serverless, well- functioning, and minimize the load` on the existing Amazon Redshift cluster. The solution should also require `minimal effort and development activity`.  
Which solution meets these requirements?

A. Unload the call center data from Amazon Redshift to Amazon S3 using an AWS Lambda function. Perform the join with AWS Glue ETL scripts.

B. Export the call center data from Amazon Redshift using a Python shell in AWS Glue. Perform the join with AWS Glue ETL scripts.

C. Create an external table using Amazon Redshift Spectrum for the call center data and perform the join with Amazon Redshift.

D. Export the call center data from Amazon Redshift to Amazon EMR using Apache Sqoop. Perform the join with Apache Hive.

### Answer - C?

这道题我的感觉是每个选项都不太对，如果一定要选的话，我更倾向于 C。

分析一下题目中的问题，存储在 S3 上的数据要与存储在 Redshift 上的数据连接，Redshift 的负载压力又很大，解决方案需要是托管的、无服务的、能够减轻 Redshift 的压力，并且不要有太多的开发环节。

解决问题的基本思路就是将连接操作挪到 Redshift 之外。题目中说到的 Glue ETL scripts、Redshift Spectrum 以及 Apache Hive 的确都可以完成。Glue ETL 和 Redshift Spectrum 都是无服务的，相比 Hive 来讲，使用与维护起来更为简单。因此首先排除选项 D。

那么如何将连接操作挪到 Redshift 之外呢？首先需要将 call center data unload 到 S3 bucket 中，然后再利用 Glue data catalog 来记录这些数据的 schema，最后通过其他工具来执行连接操作。选项 A 的问题主要在于 Lambda function 适合简单、快速的代码运行，而 UNLOAD 操作可能会耗时很长，有可能会执行失败。选项 B 中用的 Python shell 也是可行的，不过可能会需要一些开发成本。

选项 C 的含糊之处在于，call center data 原本就是存储在 Redshift 上的，而 Redshift Spectrum 只能为 S3 上存储的数据建立外部表，因此题目中说“用 Redshift Spectrum 为 call center data 建立外部表”是不可行的。但如果先把 call center data 导出到 S3，再用 Redshift Spectrum 为其创建外部表，最后再执行连接操作，却是最简便的方式。或者是为那些 csv 文件建立外部表，再执行连接操作，整个过程会更为简单。

另外再简单说一下 Redshift Spectrum 的机制，它在 S3 和 Redshift 之间加入了一个 Spectrum 层，把数据与运算独立开来（如下图所示）。数据从 S3 上读取，查询操作由 Spectrum 提交到 Redshift leader node，compute nodes 生成运算请求，再由 Spectrum 来执行这些运算，从而不占用 Redshift 计算节点的资源。
![Redshift Spectrum Architecture](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/redshift-spectrum-e7174970c87f70404f08d42d63ef5796.png)

## Q016

`#quicksight` `#permission`

A data analyst is using Amazon QuickSight for data visualization across multiple datasets generated by applications. Each application stores files within a separate Amazon S3 bucket. AWS Glue Data Catalog is used as a central catalog across all application data in Amazon S3. A new application stores its data within a separate S3 bucket. After updating the catalog to include the new application data source, the data analyst created a new Amazon QuickSight data source from an Amazon Athena table, but the import into SPICE failed.  
How should the data analyst resolve the issue?

A. Edit the permissions for the AWS Glue Data Catalog from within the Amazon QuickSight console.

B. Edit the permissions for the new S3 bucket from within the Amazon QuickSight console.

C. Edit the permissions for the AWS Glue Data Catalog from within the AWS Glue console.

D. Edit the permissions for the new S3 bucket from within the S3 console.

### Answer - B

这道题比较简单：QuickSight 是从 S3 上读取文件的，现在新添加了一个 S3 bucket，问它要如何在 QuickSight 上显示。这是一个关于 QuickSight 对 S3 bucket 的访问权限的问题，需要在 QuickSight 的控制台里修改对新的 S3 bucket 的访问权限。

另外，SPICE 指的是 Super-fast Parallel In-memory Calculation Engine，QuickSight 连接的数据集可以被导入到 SPICE 中，以提高查询操作的速度，但执行超过 30 分钟也会超时失败。

## Q017

`#kinesis-data-streams` `#kinesis-data-firehose` `#kinesis-analytics` `#amazon-sns`

A team of data scientists plans to analyze market trend data for their company's new investment strategy. The trend data comes from `five different data sources` in `large volumes`. The team `wants to utilize Amazon Kinesis` to support their use case. The team uses `SQL-like queries` to analyze trends and `wants to send notifications` based on certain significant patterns in the trends. Additionally, the data scientists `want to save the data to Amazon S3 for archival and historical re-processing`, and use AWS managed services wherever possible. The team wants to implement the `lowest-cost solution`.  
Which solution meets these requirements?

A. Publish data to one Kinesis data stream. Deploy a custom application using the Kinesis Client Library (KCL) for analyzing trends, and send notifications using Amazon SNS. Configure Kinesis Data Firehose on the Kinesis data stream to persist data to an S3 bucket.

B. Publish data to one Kinesis data stream. Deploy Kinesis Data Analytic to the stream for analyzing trends, and configure an AWS Lambda function as an output to send notifications using Amazon SNS. Configure Kinesis Data Firehose on the Kinesis data stream to persist data to an S3 bucket.

C. Publish data to two Kinesis data streams. Deploy Kinesis Data Analytics to the first stream for analyzing trends, and configure an AWS Lambda function as an output to send notifications using Amazon SNS. Configure Kinesis Data Firehose on the second Kinesis data stream to persist data to an S3 bucket.

D. Publish data to two Kinesis data streams. Deploy a custom application using the Kinesis Client Library (KCL) to the first stream for analyzing trends, and send notifications using Amazon SNS. Configure Kinesis Data Firehose on the second Kinesis data stream to persist data to an S3 bucket.

### Answer - B

这道题主要是考察 Kinesis 的几个组件的用法，题目中描述的是一个比较常见的场景：Kinesis 收集数据、进行分析、将数据存储至 S3、为特定事件发送提醒。题目中要注意的是，存储到 S3 的数据是原始数据，而不是经过分析的数据。

解决方案的架构如下图所示：
![kinesis-solution-architecture](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408133946-cd8923d84832c3ed744b8678bc1a5e57.png)

其中，Kinesis Data Streams 用于收集多个数据源的数据，Kinesis Analytics 和 Kinesis Data Firehose 分别消费 Kinesis Data Streams 中的数据。这是因为一个 Kinesis Data Streams 的数据可以被多个消费者消费，因此也就没有必要创建两个 Kinesis Data Streams 了，排除选项 C、D。至于选项 A，使用 KCL 来分析数据的话，就不能用 “SQL-like” 的查询方式了，因此排除。由以上分析可以得出，这道题应选择 B。

## Q018

`#cross-region-access` `#glue`

A company currently uses Amazon Athena to query its global datasets. The regional data is `stored in Amazon S3` in the `us-east-1 and us-west-2` Regions. The data is not encrypted. To simplify the query process and manage it centrally, the company wants to `use Athena in us-west-2 to query data from Amazon S3 in both  
Regions`. The solution should be as low-cost as possible.  
What should the company do to achieve this goal?

A. Use AWS DMS to migrate the AWS Glue Data Catalog from us-east-1 to us-west-2. Run Athena queries in us-west-2.

B. Run the AWS Glue crawler in us-west-2 to catalog datasets in all Regions. Once the data is crawled, run Athena queries in us-west-2.

C. Enable cross-Region replication for the S3 buckets in us-east-1 to replicate data in us-west-2. Once the data is replicated in us-west-2, run the AWS Glue crawler there to update the AWS Glue Data Catalog in us-west-2 and run Athena queries.

D. Update AWS Glue resource policies to provide us-east-1 AWS Glue Data Catalog access to us-west-2. Once the catalog in us-west-2 has access to the catalog in us-east-1, run Athena queries in us-west-2.

### Answer - B

题中要解决的问题是，多个数据集分别存储在两个区域里，需要在其中某个区域上用 Athena 同时访问到它们。

在 AWS 上实测了一下，选项 B 是可行的：

1. 在两个区域分别创建 S3 bucket，并各存放一个 csv 文件：![create-s3-bucket](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408153854-900dc73b45fed2c973f2a068b8fa79c9.png)
2. 创建 AWS Glue Crawler，添加以上两个 bucket 作为 data source，并运行 crawler：![create-glue-crawler](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408154252-3555dc340477b54a6253a0f41090a28f.png) ![crawler-run](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408154525-9410578cd4e9d745713e99659c368f08.png)
3. 在 Glue Table 中可以看到，两个区域内的表都被添加了：![glue-tables](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408154732-2757abe77d4fd911c7e0b0dd46e69a2a.png)
4. 用 Athena 查询两个表中的数据，均可成功：![athena-query](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/20230408154922-6522ca41d64295b6e9a78ce97dc8ccd1.png)

选项 A 要使用 DMS 迁移 Glue catalog，这是不可行的，DMS 是数据库迁移服务，并不支持数据 catalog 的迁移。选项 C 要对 us-east-1 的数据做跨域的数据复制，操作是可行的，但是 s3 的费用就会多出一倍，而且也没必要复制数据。

选项 D 提到了修改一个区域里的 Glue catalog 的 resource policy，以向另一个区域的 Glue catalog 提供访问权限。这个方式也是可行的，它可以让另一个区域的 Glue catalog 获取到该区域的 catalog 的数据，也就是库、表等定义，由此另一区域就可以访问到该区域的 S3 数据。但这个方式要复杂一些，同时需要两套 crawler 分别读取每个区域上的表，花费也会更高一些，因此排除选项 D。

## Q019

`#s3` `#redshift` `#optimization`

A large company receives files from external parties in Amazon EC2 throughout the day. At the end of the day, the files are `combined into a single file, compressed into a gzip file, and uploaded to Amazon S3`. The total size of all the files is close to `100 GB daily`. Once the files are uploaded to Amazon S3, an AWS Batch program executes a COPY command to load the files into an Amazon Redshift cluster.  
Which program modification will `accelerate the COPY process`?

A. Upload the individual files to Amazon S3 and run the COPY command as soon as the files become available.

B. Split the number of files so they are equal to a multiple of the number of slices in the Amazon Redshift cluster. Gzip and upload the files to Amazon S3. Run the COPY command on the files.

C. Split the number of files so they are equal to a multiple of the number of compute nodes in the Amazon Redshift cluster. Gzip and upload the files to Amazon S3. Run the COPY command on the files.

D. Apply sharding by breaking up the files so the distkey columns with the same values go to the same file. Gzip and upload the sharded files to Amazon S3. Run the COPY command on the files.

### Answer - B

这道题考察的是 Redshift COPY 命令的使用，即如何能够高效地将数据传输至 Redshift 集群。Redshift 使用了 Massive Parallel Processing (MPP)，也就要求处理的数据要尽可能地平均分配到每个任务资源。

Redshift 集群是由一个 Leader node 和多个 Compute nodes 组成的，每个 compute node 的资源都会被划分为多个 slice，具体 slice 的数量取决于 compute node 的类型。因此在上传数据的时候最好就能够将数据分成 slice 数量的整数倍，这样每个运算资源都可以被调动起来，并行度达到最高，效率也就最高了。因此选 B 而排除选项 C。

不选择 D 的原因是按照 distkey 的方式分割文件，不能保证分割出来的文件大小均衡，且最大程度地利用到 Redshift 的 slices。

选项 A 虽然凭直觉能看出它不合理，但原因其实讨论起来会比较复杂。

由于题目中没有具体的限定，先假设一种普通情况：单个文件占用空间不小（比如每小时都能接收一个文件，每个文件的大小就是 100GB / 24h ≈ 4GB），直接 COPY 的话可能会花费很多时间，而且因为是单个文件，不能利用到 Redshift 的 MPP，速度会很慢。

再假设有一个极端情况，单个文件很可能占用空间不太大，随随便便就可以 COPY 到 Redshift。来一个文件 COPY 一个的话，可能在这一天所有的文件都接收完的时候，COPY 也就结束了，这样岂不是连合并、压缩文件的时间都省去了。那么在这种情况下这个方案又如何呢？我认为这个方案同样不够好。从数据传输的角度来讲，传输是要按流量付费的，不加压缩的话花费自然会高一些；从文件存储的角度来讲，如果在 S3 上存放过多小文件，会影响文件的读取性能。因为每个文件都有元数据，打开关闭文件同样需要时间。这也是在任何文件系统上都不希望有过多文件的原因。

综上，排除选项 A。

## Q020

`#redshift`

A large ride-sharing company has `thousands of drivers globally` serving `millions of unique customers every day`. The company has decided to `migrate an existing data mart to Amazon Redshift`. The existing schema includes the following tables.  
✑ A `trips fact table` for information on completed rides.  
✑ A `drivers dimension table` for driver profiles.  
✑ A `customers fact table` holding customer profile information.  
The company `analyzes trip details by date and destination` to `examine profitability by region`. The `drivers data rarely changes`. The `customers data frequently changes`.  
What table design provides optimal query performance?

A. Use DISTSTYLE KEY (destination) for the trips table and sort by date. Use DISTSTYLE ALL for the drivers and customers tables.

B. Use DISTSTYLE EVEN for the trips table and sort by date. Use DISTSTYLE ALL for the drivers table. Use DISTSTYLE EVEN for the customers table.

C. Use DISTSTYLE KEY (destination) for the trips table and sort by date. Use DISTSTYLE ALL for the drivers table. Use DISTSTYLE EVEN for the customers table.

D. Use DISTSTYLE EVEN for the drivers table and sort by date. Use DISTSTYLE ALL for both fact tables.

### Answer - C

这道题主要考察的是 Redshift 上数据的 distribution style，即如何让数据分布在 Redshift 集群上以达到最优的查询性能。

Redshift 共有 4 种分布形式：

1. EVEN——均匀分布，将数据的每一行依次存储到各个计算节点以达到均匀分配的目的。这种方式适合那些没有特定查询倾向（比如不需要按照某一字段来分组或分区）、且数据量较大或者变化较大的数据，可以避免数据倾斜。均匀分布可以有效利用 Redshift 的并行运算资源。
2. ALL——全分布，每个计算节点上都存储所有的数据。适合数据量较小，且经常会被扫描到的（例如经常与其他表做连接操作）数据。它可以使运算就发生在数据所存储的节点上，以减少数据跨节点移动，从而提高效率。
3. KEY——按键分布，将数据中的某一字段指定为分布键，相同分布键的数据会存储到同一个节点上。适合那些数据量较大，查询倾向性明显的数据。因为同一分布键的数据都在同一节点上，所以分区扫描会十分高效。
4. AUTO——根据数据量大小，Redshift 会自动选择分布形式：数据量较小时，选择 ALL，数据量较大且能推断出分布键，则选 KEY，否则选 EVEN。这种方式适合那些特征不明确的数据，可以先用 ALL 来做验证。

通过以上分析，结合题目中的信息，可以看出：

1. 事实表 trips 是需要对特定区域、特定时间做查询的，且数据量比较大，因此选择用的分布形式是 KEY，用 destination 来做分布键，再在每个区域内，对日期进行排序，这样查询就会最为高效。不用日期做分布键而对区域排序的原因是，最终的结果是按区域划分的，因此要先定位到区域，使接下来对时间的过滤都集中在同一区域，从而减小数据扫描量。
2. 维度表 drivers 的数据变化很慢，且数据量很小，同时有需要经常与其他事实表做连接操作，因此将它的分布方式设置为 ALL 可以使所有的节点都可以快速地访问到该表。
3. 事实表 customers 的数据量很大，经常变化，且查询操作没有很明确的分区、分组的倾向，因此选择 EVEN 的方式，可以最大程度地利用并行运算的能力，提升性能。

因此这道题选 C。

## Q021

`#security` `#permission` `#iam`

Three teams of data analysts use `Apache Hive on an Amazon EMR cluster with the EMR File System` (EMRFS) to query data stored within each team's Amazon  
S3 bucket. The EMR cluster has `Kerberos enabled` and is configured to `authenticate users from the corporate Active Directory`. The data is highly sensitive, so access must be limited to the members of each team.
Which steps will satisfy the security requirements?

A. For the EMR cluster Amazon EC2 instances, create a service role that grants no access to Amazon S3. Create three additional IAM roles, each granting access to each team's specific bucket. Add the additional IAM roles to the cluster's EMR role for the EC2 trust policy. Create a security configuration mapping for the additional IAM roles to Active Directory user groups for each team.

B. For the EMR cluster Amazon EC2 instances, create a service role that grants no access to Amazon S3. Create three additional IAM roles, each granting access to each team's specific bucket. Add the service role for the EMR cluster EC2 instances to the trust policies for the additional IAM roles. Create a security configuration mapping for the additional IAM roles to Active Directory user groups for each team.

C. For the EMR cluster Amazon EC2 instances, create a service role that grants full access to Amazon S3. Create three additional IAM roles, each granting access to each team's specific bucket. Add the service role for the EMR cluster EC2 instances to the trust polices for the additional IAM roles. Create a security configuration mapping for the additional IAM roles to Active Directory user groups for each team.

D. For the EMR cluster Amazon EC2 instances, create a service role that grants full access to Amazon S3. Create three additional IAM roles, each granting access to each team's specific bucket. Add the service role for the EMR cluster EC2 instances to the trust polices for the base IAM roles. Create a security configuration mapping for the additional IAM roles to Active Directory user groups for each team.

### Answer - B

这道题考察了对 EMR 的权限设置，以保证只给特定的几组成员相应地权限。

首先要分析一下题目中的情景，需要哪些权限：

1. EMR 读取 S3 上的文件，需要 EMR 集群的 EC2 实例拥有 S3 的权限
2. 对于身份的认证需要将 IAM role 与 Active Directory 的用户对应起来

由于每组成员所能访问的 S3 bucket 是不同的，所以应该先为每个组创建一个 IAM role，赋予相应地 S3 权限。而除了这 3 个组之外的人不应该有访问 S3 的权限，那么需要 EMR 的 service role 默认是不带有任何 S3 的访问权限的。然后将 service role 分别添加到 3 个组各自 IAM role 的 trust policy 里，就可以使 3 个组都具有特定的 S3 访问权限并且 EC2 的实例可以 assume service role 以执行 EMR 的任务。

至于将 IAM role 与 Active Directory 的用户对应，则需创建 Security configuration mapping，将每组的 IAM role 映射至对应的用户组，这样就可以对每组的成员进行身份认证了。

由以上分析可得，此题选 B。

选项 C 和 D 在大方向上就错了，它们给了 EMR 的 Service role 对于 S3 所有的访问权限，这就意味着任何人都可以通过该 EMR 集群访问到 S3。

选项 A 与 B 的区别是，A 提出将每组的 IAM role 附到 EMR 的 service role 上，而 B 正好反过来。选项 A 的方式会使得其他人也可以使用到这个附加了 3 个组的 IAM role 的 service role，即其他人可以访问到 3 个组对应的 S3 资源，因此是不合理的。

这里要注意，EMR 的 Service role 是运行 EMR 的默认角色，它所拥有的权限应该是最小的；每个组的 IAM role 是可以被 assume 的，当使用了这个 IAM role 时就应该能访问到对应的 S3 资源。

## Q022

`#glue` `#athena`

A company is planning to create a data lake in Amazon S3. The company wants to create `tiered storage` based on access patterns and cost objectives. The solution must include `support for JDBC` connections from legacy clients, metadata management that `allows federation for access control`, and `batch-based ETL using PySpark and Scala`. `Operational management should be limited`.
Which combination of components can meet these requirements? (Choose three.)

- A. AWS Glue Data Catalog for metadata management
- B. Amazon EMR with Apache Spark for ETL
- C. AWS Glue for Scala-based ETL
- D. Amazon EMR with Apache Hive for JDBC clients
- E. Amazon Athena for querying data in Amazon S3 using JDBC drivers
- F. Amazon EMR with Apache Hive, using an Amazon RDS with MySQL-compatible backed metastore

### Answer - ACE

根据题目中的描述可以总结出几个需求点：

1. “tiered storage” 是 S3 已有的功能，可以不用关注
2. 可以访问 S3 数据且支持 JDBC 连接
   可行的方案有 Athena，Redshift Spectrum，搭建在 EC2 集群上的数据库（这里包括 EMR 集群，因为 EMR 使用的其实是 EC2 的实例）等。可以先排除最后一种情况，因为这种方式需要的手动的配置和维护有很多，不符合题中限制操作管理的要求。题目中并没有足够的信息来排除 Redshift Spectrum，但选项中没有，所以可以排除
3. 允许元数据管理 federation 的访问控制
   对这些元数据的访问是可以通过 Single-Sign On (SSO) 的方式来获得一个 IAM role。这个认证的需求由 AWS 本身提供的认证方式就可以解决，并不局限于某一个服务。而元数据管理的方案就是 Glue data catalog 或者自己搭建 Hive 服务。后者会更为复杂且难以维护。参考：[Identity and access management for AWS Glue](https://docs.aws.amazon.com/glue/latest/dg/security-iam.html#security_iam_authentication)
4. 使用 PySpark 和 Scala 的批处理 ETL
   方案是 Glue ETL 或者在 EMR 上自定义脚本，同样地，后者更为复杂。

综上，操作管理最简单的方案就是选项 A、C、E

## Q023

`#cost-effective` `#s3`

A company wants to `optimize the cost` of its data and analytics platform. The company is ingesting a number of .csv and JSON files in Amazon S3 from various data sources. Incoming data is expected to be `50 GB each day`. The company is using Amazon Athena to query the raw data in Amazon S3 directly. Most queries aggregate data from the `past 12 months`, and data that is older than 5 years is `infrequently queried`. The typical query `scans about 500 MB of data` and is `expected to return results in less than 1 minute`. The `raw data must be retained indefinitely` for compliance requirements.  
Which solution meets the company's requirements?

A. Use an AWS Glue ETL job to compress, partition, and convert the data into a columnar data format. Use Athena to query the processed dataset. Configure a lifecycle policy to move the processed data into the Amazon S3 Standard-Infrequent Access (S3 Standard-IA) storage class 5 years after object creation. Configure a second lifecycle policy to move the raw data into Amazon S3 Glacier for long-term archival 7 days after object creation.

B. Use an AWS Glue ETL job to partition and convert the data into a row-based data format. Use Athena to query the processed dataset. Configure a lifecycle policy to move the data into the Amazon S3 Standard-Infrequent Access (S3 Standard-IA) storage class 5 years after object creation. Configure a second lifecycle policy to move the raw data into Amazon S3 Glacier for long-term archival 7 days after object creation.

C. Use an AWS Glue ETL job to compress, partition, and convert the data into a columnar data format. Use Athena to query the processed dataset. Configure a lifecycle policy to move the processed data into the Amazon S3 Standard-Infrequent Access (S3 Standard-IA) storage class 5 years after the object was last accessed. Configure a second lifecycle policy to move the raw data into Amazon S3 Glacier for long-term archival 7 days after the last date the object was accessed.

D. Use an AWS Glue ETL job to partition and convert the data into a row-based data format. Use Athena to query the processed dataset. Configure a lifecycle policy to move the data into the Amazon S3 Standard-Infrequent Access (S3 Standard-IA) storage class 5 years after the object was last accessed. Configure a second lifecycle policy to move the raw data into Amazon S3 Glacier for long-term archival 7 days after the last date the object was accessed.

### Answer - A

这道题选项描述得比较复杂，用示意图来表示如下：
![das-c01-q023](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-04/das-c01-q023-ba08d447022e5f028ebdeecca3798b05.png)

题目中要解决的问题是降低成本，包括存储成本和运算成本。对于存储成本来说，数据的大小和存储时间都会造成开销。因此要尽可能地：

1. 减少数据占用的空间，采用列式存储，并进行压缩。
2. 将不常用的数据赋予更便宜的存储类型。通过对数据使用情况的分析可知，5 年前的数据是不常访问的，为了节约成本，应该把他们设置为 IA storage clas。归档的数据设置为 S3 glacier for long-term archival

对于运算成本来说，扫描的数据量越少，Athena 上的花费就越少。因此同样要选择列式存储和压缩的方式。

题目中还有一个对比的点：存储类型的更改是在文件生成的一段时间后还是在最后一次使用文件的一段时间之后。答案是在文件生成后，因为按题中描述，不常访问的数据也是有被访问到的可能性的。如果按照最后一次使用的时间来算的话，很可能好不容易等到快要 5 年了，结果这一天该数据被访问了，于是又要再等 5 年。

## Q024

`#kinesis-data-streams` `#kinesis-data-firehose` `#lambda` `#processing`

An energy company collects voltage data in `real time` from sensors that are attached to buildings. The company wants to `receive notifications` when a sequence of two voltage drops is detected within 10 minutes of a sudden voltage increase at the same building. All notifications must be delivered as quickly as possible. The system must be `highly available`. The company needs a solution that will `automatically scale` when this monitoring feature is implemented in other cities. The notification system is `subscribed to an Amazon Simple Notification Service` (Amazon SNS) topic for remediation.  
Which solution will meet these requirements?  

A. Create an Amazon Managed Streaming for Apache Kafka cluster to ingest the data. Use an Apache Spark Streaming with Apache Kafka consumer API in an automatically scaled Amazon EMR cluster to process the incoming data. Use the Spark Streaming application to detect the known event sequence and send the SNS message.

B. Create a REST-based web service by using Amazon API Gateway in front of an AWS Lambda function. Create an Amazon RDS for PostgreSQL database with sufficient Provisioned IOPS to meet current demand. Configure the Lambda function to store incoming events in the RDS for PostgreSQL database, query the latest data to detect the known event sequence, and send the SNS message.

C. Create an Amazon Kinesis Data Firehose delivery stream to capture the incoming sensor data. Use an AWS Lambda transformation function to detect the known event sequence and send the SNS message.

D. Create an Amazon Kinesis data stream to capture the incoming sensor data. Create another stream for notifications. Set up AWS Application Auto Scaling on both streams. Create an Amazon Kinesis Data Analytics for Java application to detect the known event sequence, and add a message to the message stream. Configure an AWS Lambda function to poll the message stream and publish to the SNS topic.

### Answer - A

这道题涉及到了很多 AWS 服务中比较少见的特性，答案之间的争论也比较多，需要重点注意一下这道题。

首先来梳理一下题目中要达成的目标——建立一套数据处理和分析的系统：

1. 数据来源：安装在各个建筑上的传感器，流数据
2. 分析方式：在 10 分钟的滑动窗口内，判断是否出现某种特定事件
3. 结果处理：出现该事件时，尽快触发 SNS 进行消息推送
4. 系统要求：高可用，资源自动伸缩

接下来一项项地分析实现方案。

从数据来源看，能够接收流数据的 AWS 服务有 Managed Streaming for Apache Kafka (MSK)，Kinesis Data Streams (KDS)，Kinesis Data Firehose，自己编写的运行在 EMR 上的代码，以及选项 B 中提到的利用 Amazon API Gateway 创建 RESTful API，通过 API 调用使 Lambda 将数据存储至 RDS。这里没有办法排除掉选项。

从分析方式来看，可以排除掉用 Lambda 直接进行分析的选项（选项 C），这是因为 Lambda 是无状态的，无法单独执行滑动窗口的数据分析（注：Lambda 可以对滚动窗口内的数据进行分析：[Using AWS Lambda with Amazon Kinesis - Time windows](https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis.html#services-kinesis-windows)）。另一方面，用 RDS 上的 PostgreSQL 来进行数据分析是 RDS 用法的一个反模式，即不推荐这样做，流数据的处理有更好用的工具。因此也排除选项 B。

从结果处理的方式来看，在发现该事件发生时，要尽快利用 SNS 消息通知到用户。那么选项 D 中提到的用 Lambda 方法去轮询消息流的方式是不能保证尽快发送通知的。

从系统要求来看，选项中提到的服务基本都是高可用的，尤其是 Lambda、Kinesis 系列，因为他们是无服务的，可用性很高。而 EMR 则需要手动配置，利用主副节点来提高可用性，这也是选项 A 有争议的原因之一。至于资源的自动伸缩，EMR、RDS、Lambda 都内置了自动伸缩的功能，Kinesis 需要额外的配置，才能使用 AWS  Application Auto Scaling 服务（参考 [Scale Amazon Kinesis Data Streams with AWS Application Auto Scaling](https://aws.amazon.com/blogs/big-data/scaling-amazon-kinesis-data-streams-with-aws-application-auto-scaling/)）

 通过以上分析已经可以选出答案了。最后再简单讨论一下题目中的几个服务。

1. Kinesis Analytics：的确可以进行滑动窗口内的数据分析，这是它的使用场景之一。如果不是分出来两个流并且还要用轮询的方式来获取通知，那么选项 D 的方式也是可行的。
2. KDS 与 MSK：这两个服务都适用于流数据的接入。区别在于 KDS 是全托管（Fully-managed）且无服务（Serverless）的，用户不需要关心资源的分配、幕后程序的运行机制，只需调用 AWS 接口即可；而 MSK 是由 AWS 托管的，运行在 EC2 实例上的 Kafka 应用，需要用户事先进行配置。KDS 依赖于 AWS 环境，它的程序只能运行在 AWS 平台上，但 MSK 上的程序是通用的 Kafka 应用，不依赖云平台。

## Q025

`#kinesis-data-streams` `#kinesis-data-firehose` `#kinesis-analytics`

A media company has a streaming playback application. The company needs to collect and analyze data to provide `near-real-time feedback` on playback issues `within 30 seconds`. The company requires a consumer application to identify playback issues, such as decreased quality during a specified time frame. The data will be streamed in `JSON format`. The `schema can change` over time.  
Which solution will meet these requirements?  

A. Send the data to Amazon Kinesis Data Firehose with delivery to Amazon S3. Configure an S3 event to invoke an AWS Lambda function to process and analyze the data.

B. Send the data to Amazon Managed Streaming for Apache Kafka. Configure Amazon Kinesis Data Analytics for SQL Application as the consumer application to process and analyze the data.

C. Send the data to Amazon Kinesis Data Firehose with delivery to Amazon S3. Configure Amazon S3 to initiate an event for AWS Lambda to process and analyze the data.

D. Send the data to Amazon Kinesis Data Streams. Configure an Amazon Kinesis Data Analytics for Apache Flink application as the consumer application to process and analyze the data.

### Answer - D

这道题的几个选项分成了两组，A、C 采用了 Firehose + Lambda 的方式，B、D 采用流数据工具 + Kinesis Analytics 的方式。每组的两个选项之间差别都很小。因此分析起来要格外注意。

先说选项 A、C。它们都是利用 Firehose 将数据接入到 S3，再利用 S3 的事件来触发一个 Lambda function，例如每次有新的文件存储到 S3 都会触发 Lambda 来执行分析。但题中所说的 “Configure an S3 event to invoke an AWS Lambda function” 和 “Configure Amazon S3 to initiate an event for AWS Lambda” 在我看来几乎没有什么区别。还好这两个选项都可以排除。原因是 Firehose 在接收数据的时候，首先要将数据缓存起来，当数据量达到 2MB 或者时间达到 60 秒时才会向下游写数据。因此不能满足题目中所说的 30 秒的要求。

至于选项 B、D，大体来讲也差不多，在 **[Q024]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q024)** 的解析中，我简单对比过 KDS 和 MSK 的区别，除了 KDS 有一些 shard 大小方面的限制（读：2MB/s，写：1MB/s），其他功能上都差不多。而且题目中也没有更多的要求，因此这两种服务都是适用的。问题出在 Kinesis Analytics 上，如果使用 SQL application，那么数据来源就只能是 Kinesis Data Streams / Firehose，参见 [Amazon Kinesis Data Analytics features - Kinesis Data Analytics SQL applications - Integrated Input and Output](https://aws.amazon.com/kinesis/data-analytics/features/?nc=sn&loc=2#Integrated_Input_and_Output)。因此答案只能选 D。

另外要说明的一点是，Kinesis Analytics 是通过 [Schema Discovery](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/sch-dis.html) 自动推断 JSON 数据的 schema 的。也可以通过 [Glue Schema Registry](https://docs.aws.amazon.com/glue/latest/dg/schema-registry.html) 来注册、跟踪 schema 的变化。

## Q026

`#redshift` `#s3` `#athena` `#optimization` 

An e-commerce company stores customer purchase data in Amazon RDS. The company wants a solution to store and analyze historical data. The `most recent 6 months of data will be queried frequently` for analytics workloads. This data is `several terabytes` large. `Once a month`, historical data for the `last 5 years` must be accessible and will be `joined` with the more recent data. The company wants to `optimize performance and cost`.  
Which storage solution will meet these requirements?  

A. Create a read replica of the RDS database to store the most recent 6 months of data. Copy the historical data into Amazon S3. Create an AWS Glue Data Catalog of the data in Amazon S3 and Amazon RDS. Run historical queries using Amazon Athena.

B. Use an ETL tool to incrementally load the most recent 6 months of data into an Amazon Redshift cluster. Run more frequent queries against this cluster. Create a read replica of the RDS database to run queries on the historical data.

C. Incrementally copy data from Amazon RDS to Amazon S3. Create an AWS Glue Data Catalog of the data in Amazon S3. Use Amazon Athena to query the data.

D. Incrementally copy data from Amazon RDS to Amazon S3. Load and store the most recent 6 months of data in Amazon Redshift. Configure an Amazon Redshift Spectrum table to connect to all historical data.

### Answer - D

按照题目中描述，近 6 个月的数据需要频繁访问，近 5 年的数据需每月访问。总数据量大概是几十个 TB 的大小。对于这样大的数据量，使用 RDS 来进行分析是很低效的，所以先排除选项 B。

Athena 是一个简单的交互式的 SQL 查询工具，它并不适合做大规模的数据分析。因此用它来做历史数据的连接查询同样不够高效。另外它是按照扫描的数据量来收费的，因此对于 TB 级的数据来讲，它的费用也会很高。因此排除选项 A、C。

选项 D 就是一个很标准的做法，常用数据放入 Redshift 集群，历史数据存储在 S3，这样兼顾了 Redshift 的高效和 S3 的廉价。此题选 D。

## Q027

`#athena`

A company leverages Amazon Athena for ad-hoc queries against data stored in Amazon S3. The company wants to implement additional controls to `separate query execution and query history` among users, teams, or applications running in the same AWS account to comply with internal security policies.  
Which solution meets these requirements?  

A. Create an S3 bucket for each given use case, create an S3 bucket policy that grants permissions to appropriate individual IAM users. and apply the S3 bucket policy to the S3 bucket.

B. Create an Athena workgroup for each given use case, apply tags to the workgroup, and create an IAM policy using the tags to apply appropriate permissions to the workgroup.

C. Create an IAM role for each given use case, assign appropriate permissions to the role for the given use case, and add the role to associate the role with Athena.

D. Create an AWS Glue Data Catalog resource policy for each given use case that grants permissions to appropriate individual IAM users, and apply the resource policy to the specific tables used by Athena.

### Answer - B

这是一道基础题，题目中描述的需求正是 Athena workgroup 的功能。给每个用例创建一个 workgroup，可以将用例间的查询命令、查询历史以及开销分隔开来。参考 [Separate queries and managing costs using Amazon Athena workgroups](https://aws.amazon.com/blogs/big-data/separating-queries-and-managing-costs-using-amazon-athena-workgroups/)。因此选 B。

而对于其他选项，不论是给 S3 bucket 设置 policy，还是创建 IAM role，或是给 Glue Data Catalog 设置 resource policy，都只是限制了 Athena 可访问到的数据，但无法将 Query 的执行和历史区分开。也就是说在运行同一条查询的时候，有的用户会成功，有的用户会因为没有数据访问权限而失败，但这些查询都会被记录下来，所有人都可以看到。

## Q028

`#quicksight`

A company wants to use an automatic machine learning (ML) `Random Cut Forest` (RCF) algorithm to visualize complex real-world scenarios, such as detecting seasonality and trends, excluding outliers, and imputing missing values.  
The team working on this project is `non-technical` and is looking for an out-of-the-box solution that will require the `LEAST amount of management` overhead.  
Which solution will meet these requirements?  

A. Use an AWS Glue ML transform to create a forecast and then use Amazon QuickSight to visualize the data.

B. Use Amazon QuickSight to visualize the data and then use ML-powered forecasting to forecast the key business metrics.

C. Use a pre-build ML AMI from the AWS Marketplace to create forecasts and then use Amazon QuickSight to visualize the data.

D. Use calculated fields to create a new forecast and then use Amazon QuickSight to visualize the data.

### Answer - B

这道题也是一道基础题。题目要求非技术人员使用 ML 对数据进行分析，这正好就是 QuickSight 的领域。QuickSight 内置了很多 ML 方案，其中就包括 Random Cut Forest。参考 [Gaining insights with machine learning (ML) in Amazon QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/making-data-driven-decisions-with-ml-in-quicksight.html)。

其他方案里，选项 A 的 [Glue ML Transform](https://docs.aws.amazon.com/glue/latest/dg/machine-learning.html) 是用于 ETL 的转换步骤中的，目前只有查找匹配项（即数据中是否有相同行）这一功能。选项 C、D 都可行，但需要技术能力的支持，不能满足 “LEAST amount of management” 的要求。

## Q029

`#quicksight` `#security` `#permission`

A retail company's data analytics team recently created `multiple product sales analysis dashboards` for the average selling price per product using Amazon  
QuickSight. The dashboards were created `from .csv files uploaded to Amazon S3`. The team is now planning to share the dashboards with the `respective external product owners` by creating individual users in Amazon QuickSight. For compliance and governance reasons, `restricting access` is a key requirement. The product owners `should view only their respective product analysis` in the dashboard reports.  
Which approach should the data analytics team take to allow product owners to view only their products in the dashboard?  

A. Separate the data by product and use S3 bucket policies for authorization.

B. Separate the data by product and use IAM policies for authorization.

C. Create a manifest file with row-level security.

D. Create dataset rules with row-level security.

### Answer - D

这也是一道很基础的题目，重点是要读清题目要求。一个零售公司对其产品的销售情况做了一个分析面板，想要把它分享给产品的供应商，并且不同产品的数据只能由该产品的供应商访问。由于各产品的数据都混杂在 csv 文件中，所以通过 S3 bucket policy 或者 IAM policy 是不能区分访问权限的。选项 C 中提到的 manifest file 是存储在 S3 bucket 里的特殊文件，用来提示 QuickSight 需要导入哪些 S3 bucket 中的文件（参考 [Supported formats for Amazon S3 manifest files](https://docs.aws.amazon.com/quicksight/latest/user/supported-manifest-file-format.html)），同样只能约束 QuickSight 可访问的文件，但不具备行级别的控制，不能约束哪些人能访问哪些数据。所以排除选项 A、B、C。

这样的细粒度的访问控制正好可以用 QuickSight 提供的 row-level security 来实现，QuickSight 可以控制每一行数据能够被谁访问。参考 [Using row-level security (RLS) in Amazon QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/row-level-security.html)。

## Q030

`#emr` `#lambda` `#glue` `#cost-effective`

A company has developed an Apache Hive script to `batch process data stored in Amazon S3`. The script needs to `run once every day and store the output in  
Amazon S3`. The company tested the script, and it completes within 30 minutes on a small local three-node cluster.  
Which solution is the `MOST cost-effective` for scheduling and executing the script?  

A. Create an AWS Lambda function to spin up an Amazon EMR cluster with a Hive execution step. Set KeepJobFlowAliveWhenNoSteps to false and disable the termination protection flag. Use Amazon CloudWatch Events to schedule the Lambda function to run daily.

B. Use the AWS Management Console to spin up an Amazon EMR cluster with Python, Hue, Hive, and Apache Oozie. Set the termination protection flag to true and use Spot Instances for the core nodes of the cluster. Configure an Oozie workflow in the cluster to invoke the Hive script daily.

C. Create an AWS Glue job with the Hive script to perform the batch operation. Configure the job to run once a day using a time-based schedule.

D. Use AWS Lambda layers and load the Hive runtime to AWS Lambda and copy the Hive script. Schedule the Lambda function to run daily by creating a workflow using AWS Step Functions.


### Answer - A

这道题是有陷阱的，读完题目首先想到的就是 Glue job 不就是用来做这个的嘛，既能处理数据，又能设置定时调度。然而题目要求的是运行 Hive script，这是 Glue job 不支持的。Glue job 只支持 Python、Spark 脚本。因此排除选项 C。

那么就需要其他运行 Hive 脚本的方式了，EMR 和 Lambda 都是备选方案。

对于 Lambda 来说，本身没有提供 Hive 的 runtime，那么就需要用户提供自定义的 runtime 或者是容器镜像，多少会有些麻烦。同时由于该 Hive 脚本在本地环境运行需要 30 分钟左右，那么迁移到 Lambda 上来运行就会超时（Lambda 运行时间最多 15 分钟）。因此排除选项 D 。

选项 A 和 B 都是利用 EMR 运行 Hive 脚本，区别在于选项 B 中的调度系统（Oozie）是运行在 EMR 上的，那就意味着这个 EMR 集群必须一直运行，否则任务调度就失效了。这样会产生很多不必要的开销，因此排除选项 B。像选项 A 那样，用 CloudWatch 生成 Event 来触发 Lambda function，再由 Lambda function 来启动 EMR 集群来运行脚本的方式是最经济的。参考 [Tutorial: Schedule AWS Lambda Functions Using CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/RunLambdaSchedule.html)。

## Q031

`#redshift` `#optimization`

A company wants to `improve the data load time` of a sales data dashboard. Data has been collected as `.csv files and stored within an Amazon S3 bucket that is partitioned by date`. The data is then `loaded to an Amazon Redshift` data warehouse for `frequent analysis`. The data volume is up to `500 GB per day`.  
Which solution will improve the data loading performance?  

A. Compress .csv files and use an INSERT statement to ingest data into Amazon Redshift.

B. Split large .csv files, then use a COPY command to load data into Amazon Redshift.

C. Use Amazon Kinesis Data Firehose to ingest data into Amazon Redshift.

D. Load the .csv files in an unsorted key order and vacuum the table in Amazon Redshift.

### Answer - B

这道题的需求比较简单，就是要将每天 500 GB 的数据从 S3 bucket 传输到 Redshift。在 **[Q019]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q019)** 的解析中说明了，Redshift 的 MPP 适合进行并行数据处理，此题中的数据传输正需要利用这一特性。因此拆分大文件，让 Redshift 并行地加载它们是最快速的方式。不过这里要说明的是，其实 Redshift 的  COPY 操作本身就会尝试分割 128 MB 或者更大的文件，对于普通 CSV 文件、bzip 压缩的 CSV 文件以及 ORC 和 Parquet 文件，Redshift 都可以自动分割。其他不能自动分割的文件，才会推荐用户手动分割。所以其实选项 B 的做法没有错，但多少有些累赘。参考 [Loading data files](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-use-multiple-files.html)。

选项 A 中，首先 INSERT 不适合做大规模数据的插入，参考 [Amazon Redshift - INSERT - Note](https://docs.aws.amazon.com/redshift/latest/dg/r_INSERT_30.html#Note:~:text=INSERT%20INTO...SELECT%29.-,Note,-We%20strongly%20encourage)。其次压缩 csv 文件会引入额外的处理时间。

选项 C 使用 Kinesis Firehose 是可行的，但是对于提升传输速度没有帮助，因为这样的传输是串行的。

选项 D 在传输方面和 C 一样，对于速度提升没有什么帮助。值得一提的是 `VACUUM` 操作。`VACCUM` 会将 Redshift 中的指定表进行排序并且回收空间。但实际上 Redshift 本身就会自动地在后台执行 `VACUUM DELETE` 操作。参考 [Amazon Redshift - VACUUM](https://docs.aws.amazon.com/redshift/latest/dg/r_VACUUM_command.html)。

## Q032

`#redshift` `#optimization`

A company has a data warehouse in Amazon Redshift that is approximately 500 TB in size. New data is `imported every few hours` and `read-only queries are run throughout the day and evening`. There is a `particularly heavy load` with no writes for several hours each morning on business days. During those hours, some queries are queued and take a long time to execute. The company needs to optimize query execution and `avoid any downtime`.  
What is the `MOST cost-effective` solution?  

A. Enable concurrency scaling in the workload management (WLM) queue.

B. Add more nodes using the AWS Management Console during peak hours. Set the distribution style to ALL.

C. Use elastic resize to quickly add nodes during peak times. Remove the nodes when they are not needed.

D. Use a snapshot, restore, and resize operation. Switch to the new target cluster.

### Answer - A

这道题的重点是分析出性能问题的原因。由题目描述可知，每个工作日的早晨都会几个小时的负载过重，导致查询指令要排队很久才能被执行。那么为什么查询指令会进入队列等待呢？这是因为 Redshift 要为每个查询分配资源，例如内存空间。设计一个 query queue 可以让多个查询指令易于管理。这个性能问题的直接原因就是在这几个小时内，那些只读的查询指令被其他长时间的查询操作阻碍了，于是它们分配不到资源。

解决资源问题，最直观的方式肯定是直接增加集群中的节点，但往往都会造成很多额外的开销。运行中的 Redshift 是无法无缝增减节点的，只能通过 RESIZE 操作。不论是 elastic resize 还是 classic resize 都会造成集群在一段时间内不可用。elastic resize 需要几分钟，classic resize 则需几小时到几天。因此选项 B、C、D 都不满足 “avoid any downtime” 的要求。故而选 A。

选项 A 中提到了启用 WLM 的 concurrency scaling 功能。首先说 WLM，它可以创建多个 query queue，不同优先级的查询可以进入不同的队列。如题中的这种情况，耗时短的查询操作可以不被耗时长的查询阻碍，直接分配到资源从而得以运行。而 concurrency scaling 可以给现有的集群增加额外的集群容量。把两者结合起来，高优先级的查询会经由 WLM 的队列分配给额外的 concurrency-scaling cluster 单独运行，由此解决了题中出现的性能问题。参考 [Working with concurrency scaling](https://docs.aws.amazon.com/redshift/latest/dg/concurrency-scaling.html)。

## Q033

`#redshift`

A company analyzes its data in an Amazon Redshift data warehouse, which currently has a cluster of `three dense storage nodes`. Due to a recent business acquisition, the company `needs to load an additional 4 TB of user data` into Amazon Redshift. The engineering team will combine `all the user data` and apply complex calculations that require `I/O intensive` resources. The company needs to `adjust the cluster's capacity` to support the change in analytical and storage requirements.
Which solution meets these requirements?

A. Resize the cluster using elastic resize with dense compute nodes.

B. Resize the cluster using classic resize with dense compute nodes.

C. Resize the cluster using elastic resize with dense storage nodes.

D. Resize the cluster using classic resize with dense storage nodes.

### Answer - A

首先要明确 dense storage 节点和 dense compute 节点的区别。Dense storage 节点的存储资源是要优先于计算资源的，硬盘空间是 TB 级的，适用于大数据量的场景；Dense compute 节点则相反，具有更多的计算资源，适用于高运算强度的场景。

那么从此题的需求来看，I/O intensive 就指示着需要更多的计算资源，也就是说节点类型应该变更为 dense compute，因此排除选项 C 和 D。

再来区分一下 elastic resize 和 classic resize。Elastic resize 可以在不停机的同时扩展集群及其资源、更改节点类型。此外，它还能够在保留现有数据存储容量的同时扩展集群的计算容量。Classic resize 需要停机时间并替换节点，而不只是调整大小。 因此可能导致数据丢失并且需要更长的维护时间。

综上，应该选择 A。参考 [Overview of managing clusters in Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-operations.html)。

## Q034

`#emr` `#security`

A company stores its sales and marketing data that includes personally identifiable information (PII) in Amazon S3. The company allows its analysts to launch their own Amazon EMR cluster and run analytics reports with the data. To meet compliance requirements, the company must ensure the `data is not publicly accessible` throughout this process. A data engineer has secured Amazon S3 but must ensure the individual EMR clusters created by the analysts are not exposed to the public internet.
Which solution should the data engineer to meet this compliance requirement with `LEAST amount of effort`?

A. Create an EMR security configuration and ensure the security configuration is associated with the EMR clusters when they are created.

B. Check the security group of the EMR clusters regularly to ensure it does not allow inbound traffic from IPv4 0.0.0.0/0 or IPv6 ::/0.

C. Enable the block public access setting for Amazon EMR at the account level before any EMR cluster is created.

D. Use AWS WAF to block public internet access to the EMR clusters across the board.

### Answer - C

首先要明确题中的要求只是数据不会被外部访问，而并非要在不同的分析人员间设置访问权限。这其实是一个较高层面的、粗粒度的访问控制。那么只要在账户级别——即公司内部——设置一个禁止公共访问的策略即可。正如选项 C 所描述的那样，EMR 为每个账户的每个区域都提供了一个默认的配置——block public access，这样除非特意地用安全策略设置公网访问，任意新建的 EMR 集群都会应用这个配置。特别要说明的是，这一规则只会在创建集群时才被应用，当集群运行起来后，拥有适当权限的 IAM role 是可以修改安全规则以使集群在公网可访问的。参考 [Using Amazon EMR block public access](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-block-public-access.html)。

选项 A 和 B 的问题在于：

1. 粒度太细。按照上面分析可知，这个需求只需要在整个账户的层面设置即可
2. 每个集群都需要设置，选项 B 甚至还需要定期检查，这样很不方便，增加了维护成本。

选项 D 中提到的 [WAF - Web Application Firewall](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-aws-waf.html) 就和题中的需求完全无关了。它主要是用来保护网络应用的，更具体地说是通过限制对 API 的请求，来阻止网络攻击的。它可以过滤特定 IP、网域、区域等传来的请求，也可过滤包含特定内容的请求。它属于 [AWS API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) 的一部分。

## Q035

`#redshift` `#optimization`

A financial company uses Amazon S3 as its data lake and has set up a data warehouse using a `multi-node Amazon Redshift cluster`. The data files in the data lake are `organized in folders based on the data source` of each data file. All the data files are loaded to one table in the Amazon Redshift cluster using a separate  
COPY command for each data file location. With this approach, loading all the data files into Amazon Redshift takes a long time to complete. Users want a `faster solution with little or no increase in cost while maintaining the segregation` of the data files in the S3 data lake.  
Which solution meets these requirements?  

A. Use Amazon EMR to copy all the data files into one folder and issue a COPY command to load the data into Amazon Redshift.

B. Load all the data files in parallel to Amazon Aurora, and run an AWS Glue job to load the data into Amazon Redshift.

C. Use an AWS Glue job to copy all the data files into one folder and issue a COPY command to load the data into Amazon Redshift.

D. Create a manifest file that contains the data file locations and issue a COPY command to load the data into Amazon Redshift.

### Answer - D

这是一道基础题，考察 Redshift COPY 命令的最佳实践。选项 A、B、C 本质上都是先把不同目录下（在 S3 的语境里可以称为前缀）的数据汇集到一起，再将它们导入到 Redshift 表中。这些方案都是能够实现数据导入的，但是都会引入额外的成本。选项 A、C 略比选项 B 强一点，因为至少它们的 COPY 操作可以是并行的，而选项 B 使用 Glue job 来导入数据，这个过程就变成了串行的，反而降低了效率。

选项 D 就是一个最佳的做法。Menifest 文件是一个 JSON 格式的文本文件，记录着所有需要导入 Redshift 的文件的绝对路径。Redshift 通过解析这个文件，就可以自动地并行地使用 COPY 命令导入数据。这样既不破坏 S3 上文件的分隔，也不引入新的服务，数据传输速度还能变快。参考 [Using a manifest to specify data files](https://docs.aws.amazon.com/redshift/latest/dg/loading-data-files-using-manifest.html)。

## Q036

`#redshift`

A company's marketing team has asked for help in identifying a `high performing long-term storage service` for their data based on the following requirements:  
✑ The data size is approximately `32 TB` uncompressed.  
✑ There is a `low volume of single-row inserts` each day.  
✑ There is a `high volume of aggregation queries` each day.  
✑ `Multiple complex joins` are performed.  
✑ The queries typically `involve a small subset of the columns` in a table.  
Which storage service will provide the MOST performant solution?  

A. Amazon Aurora MySQL

B. Amazon Redshift

C. Amazon Neptune

D. Amazon Elasticsearch

### Answer - B

这是目前遇到的最基础的题目了。

首先从数据量就排除了选项 A，更别说 MySQL 并不适用于 OLAP。

Amazon Neptune 是一个图（Graph）数据库，正如 Graph 的特点，数据间是高度连接的，这种图数据用普通的关系型数据库或者数据仓库中的维度建模来表示是很不现实的。当然在题目的要求下，这些数据也不是图数据。

ElasticSearch（现在叫 OpenSearch）是一个搜索引擎，主要用于全文本搜索。并不适用于题目所述的场景。

从题目要求来看，数据是列式存储的，有着大量的聚合、连接运算，数据量大，这就是典型的数据仓库的场景，使用 Redshift 正合适。

## Q037

`#kinesis-data-firehose` `#opensearch` `#quicksight`

A technology company is creating a dashboard that will `visualize and analyze time-sensitive data`. The data will come in through Amazon Kinesis Data Firehose with the `buffer interval set to 60 seconds`. The dashboard `must support near-real-time data`.
Which visualization solution will meet these requirements?

A. Select Amazon OpenSearch Service (Amazon Elasticsearch Service) as the endpoint for Kinesis Data Firehose. Set up an OpenSearch Dashboards (Kibana) using the data in Amazon OpenSearch Service (Amazon ES) with the desired analyses and visualizations.

B. Select Amazon S3 as the endpoint for Kinesis Data Firehose. Read data into an Amazon SageMaker Jupyter notebook and carry out the desired analyses and visualizations.

C. Select Amazon Redshift as the endpoint for Kinesis Data Firehose. Connect Amazon QuickSight with SPICE to Amazon Redshift to create the desired analyses and visualizations.

D. Select Amazon S3 as the endpoint for Kinesis Data Firehose. Use AWS Glue to catalog the data and Amazon Athena to query it. Connect Amazon QuickSight with SPICE to Athena to create the desired analyses and visualizations.

### Answer - A

基础题，考察近实时数据的分析与可视化。AWS 的服务中只有 OpenSearch 可以做到这一点，它可以与 Kinesis Data Firehose 集成，快速地处理经由 Firehose 传入的数据。因此选 A。

选项 B 中提到的 SageMaker Jupyter notebook 更适合做数据探索。而且有 Firehose 传数据到 S3 再将数据读入 Jupyter notebook 会花费更长时间。

同理，选项 C、D 都用到了太多其他的服务，比如 Firehose -> Redshift -> QuickSight SPICE，其中每一步都涉及到数据传输。而且 QuickSight 上的展示的数据需要定时刷新，时间间隔至少为每小时（企业版，普通版至少是每天）。参考 [Refreshing SPICE data](https://docs.aws.amazon.com/quicksight/latest/user/refreshing-imported-data.html)。

## Q038

`#ecr` `#optimization`

A financial company uses `Apache Hive on Amazon EMR` for ad-hoc queries. Users are complaining of sluggish performance.
A data analyst notes the following:
✑ Approximately `90% of queries are submitted 1 hour after the market opens`.
Hadoop Distributed File System `(HDFS) utilization never exceeds 10%`.

Which solution would help address the performance issues?

A. Create instance fleet configurations for core and task nodes. Create an automatic scaling policy to scale out the instance groups based on the Amazon CloudWatch CapacityRemainingGB metric. Create an automatic scaling policy to scale in the instance fleet based on the CloudWatch CapacityRemainingGB metric.

B. Create instance fleet configurations for core and task nodes. Create an automatic scaling policy to scale out the instance groups based on the Amazon CloudWatch YARNMemoryAvailablePercentage metric. Create an automatic scaling policy to scale in the instance fleet based on the CloudWatch YARNMemoryAvailablePercentage metric.

C. Create instance group configurations for core and task nodes. Create an automatic scaling policy to scale out the instance groups based on the Amazon CloudWatch CapacityRemainingGB metric. Create an automatic scaling policy to scale in the instance groups based on the CloudWatch CapacityRemainingGB metric.

D. Create instance group configurations for core and task nodes. Create an automatic scaling policy to scale out the instance groups based on the Amazon CloudWatch YARNMemoryAvailablePercentage metric. Create an automatic scaling policy to scale in the instance groups based on the CloudWatch YARNMemoryAvailablePercentage metric.

### Answer - D

从选项中可以看出来，重点是区分 “instance fleet vs instance group” 以及 CloudWatch 的指标 “CapacityRemainingGB vs YARNMemoryAvailablePercentage”。

Instance fleet 允许 EMR 集群使用不同类型、不同可用区的实例；而 instance group 内的实例只能是同一类型的。关于这一部分内容，在创建 EMR 集群时可以有更直观的感受，建议亲自动手创建一下集群，就会看到有不同的选项和配置了。参考 [Create a cluster with instance fleets or uniform instance groups](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-instance-group-configuration.html)。

CapacityRemainingGB 指的是 HDFS 上剩余的空间；YARNMemoryAvailablePercentage 指的是 YARN 集群上可用的内存空间的百分比。

题目要求是定位到造成性能问题的原因。四个选项都采用的是利用 CloudWatch 指标来自动动态扩充集群容量。那么按照条件，接下来分析一下应该使用哪种指标。

从题目描述可以看出，HDFS 的使用量从来没超过 10%，就说明存储空间不是性能瓶颈，因此 CapacityRemainingGB 是没有用的。造成性能问题的原因很可能是查询量突然增大引起的可用内存不足，所以可以追踪 YARNMemoryAvailablePercentage 以验证猜想。所以排除选项 A、C。

不选 A 的原因是 instance  fleet 不支持 automatic scaling，参考 [Using automatic scaling with a custom policy for instance groups](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-automatic-scaling.html):

> Automatic scaling with a custom policy is available with the instance groups configuration and is not available when you use instance fleets.

需要说明的是，这道题的背景可能是较早版本的 AWS EMR 服务，当时的 instance fleet 是不支持 automatic scaling 的。但现在的版本已经将 automatic scaling 改成了 EMR managed scaling。EMR managed scaling 可以不需要用户提前预测集群的使用容量，不需要手动编写规则，而是利用内置的算法，自动地检测集群运行的情况，自动地伸缩集群容量。它和 automatic scaling 的区别见 [EMR Managed Scaling vs. Auto Scaling](https://aws.amazon.com/blogs/big-data/introducing-amazon-emr-managed-scaling-automatically-resize-clusters-to-lower-cost/#:~:text=emr%20managed%20scaling%20vs.%20auto%20scaling)。

## Q039

`#emr` `#s3` `#optimization`

A media company has been performing analytics on log data generated by its applications. There has been a recent increase in the number of concurrent analytics jobs running, and the overall performance of existing jobs is decreasing as the number of new jobs is increasing. `The partitioned data is stored in Amazon S3 One Zone-Infrequent Access` (S3 One Zone-IA) and the analytic processing is performed on Amazon EMR clusters using the `EMR File System (EMRFS) with consistent view enabled`. A data analyst has determined that `it is taking longer for the EMR task nodes to list objects` in Amazon S3.
Which action would MOST likely increase the performance of accessing log data in Amazon S3?

A. Use a hash function to create a random string and add that to the beginning of the object prefixes when storing the log data in Amazon S3.

B. Use a lifecycle policy to change the S3 storage class to S3 Standard for the log data.

C. Increase the read capacity units (RCUs) for the shared Amazon DynamoDB table.

D. Redeploy the EMR clusters that are running slowly to a different Availability Zone.

### Answer - C

这道题也是一个较早期的的题目，Amazon S3 早已支持了 strongly consistent，所以 EMRFS 上不再需要设置 consistent view 来保证文件的一致性（在 **[Q003]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q003)** 中已提到过）。不过仍然可以分析一下这道题目。题目的前提是，EMRFS 的 consistent view 已启用，日志文件在 S3 的存储类型是 One Zone - Infrequent。

题中已明确指出，造成性能问题的原因是 EMR 在对 S3 执行 *list objects* 操作时所需时间越来越长。那么 list objects 操作需要经过哪些过程呢？根据 EMRFS consistent view 的特性，不难推测出，EMR 首先需要获取到 S3 上的文件列表，再去 DynamoDB 中查看文件的元数据，然后对比元数据和 S3 上文件的差别，判断是否已经是一致的，最后才能列出该文件。这里的瓶颈就在于读取 DynamoDB 的元数据，因为 DynamoDB 是有性能限制的，读操作消耗 RCU，如果 RCU 设置得太小，则会使读性能受限。因此增加 RCU 是有可能改善整个操作性能的。

选项 A 是优化 S3 的并行度的，一个 prefix 每秒支持 5500 次读请求，如果添加前缀的数量，就可以同时进行更多次的读请求。但在 EMR consistent view 下，即使读到了数据，依然需要对比 DynamoDB 中的元数据，不提高 RCU，性能依旧提升不了。

选项 B 中提到了 S3 的存储类型，这些存储类型主要是改变了文件的可访问性从而减少不常访问的文件所需的费用。选项 D 与题中的性能没有直接关系。

## Q040

`#glue`

A company has developed several AWS Glue jobs to `validate and transform its data` from Amazon S3 and load it into Amazon RDS for MySQL `in batches once every day`. The ETL jobs read the S3 data using a DynamicFrame. Currently, the ETL developers are experiencing challenges in `processing only the incremental data on every run`, as the AWS Glue job processes all the S3 input data on each run.
Which approach would allow the developers to solve the issue with minimal coding effort?

A. Have the ETL jobs read the data from Amazon S3 using a DataFrame.

B. Enable job bookmarks on the AWS Glue jobs.

C. Create custom logic on the ETL jobs to track the processed S3 objects.

D. Have the ETL jobs delete the processed objects or data from Amazon S3 after each run.

### Answer - B

基础题，Glue 的 job bookmarks 就是用来记录之前的执行情况的，这样新一轮的运行就可以在之前的基础上进行，达到增量的目的。关于 job bookmarks，在 **[Q005]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q005)** 中也已提到过。

选项 A、C，是可以实现的，但是代码量会很多，不符合要求。

选项 D 肯定是不推荐的，数据工程中，尽量不选择删除历史文件。

## Q041

`#dynamodb` `#redshift` `#security`

A mortgage company has a microservice for accepting payments. This microservice uses the Amazon DynamoDB `encryption client with AWS KMS managed keys` to encrypt the sensitive data before writing the data to DynamoDB. The finance team should be able to `load this data into Amazon Redshift` and aggregate the values within the sensitive fields. The Amazon Redshift cluster is shared with other data analysts from different business units.
Which steps should a data analyst take to accomplish this task efficiently and securely?

A. Create an AWS Lambda function to process the DynamoDB stream. Decrypt the sensitive data using the same KMS key. Save the output to a restricted S3 bucket for the finance team. Create a finance table in Amazon Redshift that is accessible to the finance team only. Use the COPY command to load the data from Amazon S3 to the finance table.

B. Create an AWS Lambda function to process the DynamoDB stream. Save the output to a restricted S3 bucket for the finance team. Create a finance table in Amazon Redshift that is accessible to the finance team only. Use the COPY command with the IAM role that has access to the KMS key to load the data from S3 to the finance table.

C. Create an Amazon EMR cluster with an EMR_EC2_DefaultRole role that has access to the KMS key. Create Apache Hive tables that reference the data stored in DynamoDB and the finance table in Amazon Redshift. In Hive, select the data from DynamoDB and then insert the output to the finance table in Amazon Redshift.

D. Create an Amazon EMR cluster. Create Apache Hive tables that reference the data stored in DynamoDB. Insert the output to the restricted Amazon S3 bucket for the finance team. Use the COPY command with the IAM role that has access to the KMS key to load the data from Amazon S3 to the finance table in Amazon Redshift.

### Answer - A

题目要求是从 DynamoDB 加载带有敏感信息的数据到 Redshift，同时这些数据只能被 finance team 访问。

首先要确定的是数据是如何进入 Redshift 的：DynamoDB 上的数据是被加密的，要么在读取数据时对其解密，要么就利用 Redshift 的 COPY 操作直接写入被加密的数据。由题中描述可知，该公司使用的是 client-side encryption with KMS key (CSE-KMS)，而 Redshift 的 COPY 操作不支持这种方式加密的数据。因此只能选择在读取 DynamoDB 的时候就对数据解密。因此排除选项 B、D。

选项 C（以及选项 D）的问题在于引入了 EMR 来处理数据，这其实是不必要的。因为对这些数据的处理只涉及到解密的过程，这个需求使用 Lambda 就可实现。EMR with Hive 会将问题复杂化。

而选项 A 就很好地解决了问题，用 Lambda 对 DynamoDB 上的数据进行解密并存储至专有的 S3 bucket，Redshift 上也创建了专有的表，这就保证了数据的私密性。最后利用 COPY 操作写入数据，从而实现需求。

## Q042

`#glue`

A company is building a data lake and needs to `ingest data from a relational database` that has `time-series data`. The company wants to `use managed services` to accomplish this. The process needs to be scheduled daily and bring incremental data only from the source into Amazon S3.
What is the MOST cost-effective approach to meet these requirements?

A. Use AWS Glue to connect to the data source using JDBC Drivers. Ingest incremental records only using job bookmarks.

B. Use AWS Glue to connect to the data source using JDBC Drivers. Store the last updated key in an Amazon DynamoDB table and ingest the data using the updated key as a filter.

C. Use AWS Glue to connect to the data source using JDBC Drivers and ingest the entire dataset. Use appropriate Apache Spark libraries to compare the dataset, and find the delta.

D. Use AWS Glue to connect to the data source using JDBC Drivers and ingest the full data. Use AWS DataSync to ensure the delta only is written into Amazon S3.

### Answer - A

这道题比较简单，思路和 **[Q040]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q040)** 一样。其他的几个选项都太过麻烦，使用的服务也很复杂。关于 Data Sync，在 **[Q013]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q013)** 中也提到过，更适用于一次性的、不同文件系统间的、文件的迁移。

## Q043

`#redshift` `#security`

An Amazon Redshift database contains sensitive user data. Logging is necessary to meet compliance requirements. The logs `must contain database authentication attempts, connections, and disconnections`. The logs must also `contain each query run against the database and record which database user ran each query`.
Which steps will create the required logs?

A. Enable Amazon Redshift Enhanced VPC Routing. Enable VPC Flow Logs to monitor traffic.

B. Allow access to the Amazon Redshift database using AWS IAM only. Log access using AWS CloudTrail.

C. Enable audit logging for Amazon Redshift using the AWS Management Console or the AWS CLI.

D. Enable and download audit reports from AWS Artifact.

### Answer - C

从题目要求来看，需要记录的都是所有针对 Redshift 数据库的操作：认证、连接、查询等等。这些操作只能通过 Audit logging 来记录。选项 A 中提到的 VPC flow logs 只能记录访问该 VPC 的网络流量；选项 B 中的 CloudTrail 用来记录 Redshift 集群的配置的修改，例如集群容量的增减、安全组的修改等；选项 D 中的 AWS Artifact 提供按需下载AWS安全和合规性文档，与日志记录没有关系。

## Q044

`#kinesis-data-streams` `#optimization`

A company that monitors weather conditions from remote construction sites is setting up a solution to collect temperature data from the following two weather stations.
✑ Station A, which has 10 sensors
✑ Station B, which has five sensors
These weather stations were placed by onsite subject-matter experts.
`Each sensor has a unique ID`. The data collected from each sensor will be collected using Amazon Kinesis Data Streams.
Based on the total incoming and outgoing data throughput, a single Amazon Kinesis data stream with two shards is created. Two partition keys are created `based on the station names`. During testing, there is a bottleneck on data coming from Station A, but not from Station B. Upon review, it is confirmed that the `total stream throughput is still less than the allocated Kinesis Data Streams throughput`.
How can this bottleneck be resolved without increasing the overall cost and complexity of the solution, while retaining the data collection quality requirements?

A. Increase the number of shards in Kinesis Data Streams to increase the level of parallelism.

B. Create a separate Kinesis data stream for Station A with two shards, and stream Station A sensor data to the new stream.

C. Modify the partition key to use the sensor ID instead of the station name.

D. Reduce the number of sensors in Station A from 10 to 5 sensors.

### Answer - C

这道题是一个很好地关于 Kinesis Data Stream 性能调优的例子。题中的场景也是很典型的。

首先来分析一下问题是怎么产生的。数据来自于两个天气站，每个站点的数据进入一个 shard。但是由于站点 A 的传感器多、数据量大，所以一个 shard 处理不过来，导致了瓶颈的产生。从后面的描述也可看出，整个 stream 的吞吐量还没有达到实际分配的量，也就是说整体的性能没跑满，而单个 shard 又爆表了，这就是所谓的 “hot shard”。

那么这个 hot shard 是如何产生的？原因是数据是按站点名分区的，数据量大的分区自然就 “hot” 了。所以最简单的解决方案就是让数据尽可能平均地分布给每个 shard，另外，对于这种分区的数据，数据倾斜是很常见且需妥善处理的问题。按照 sensor ID 来进行分区会让数据分布平均。

选项 A 提到的增加 shard 数量的确可以减少 hot shard 的负荷，但是它只会使每个 shard 的利用率更低，总吞吐量更加达不到额定值，于是造成更多地浪费。

选项 B 是要新建一个 stream，排除它的原因和上面类似，花费更多，效率更低。

选项 D 是要减少站点 A 的传感器数量，这也是不推荐的，这样会影响到业务，比如数据分析的效果。在解决问题的时候不要影响原本的业务功能是最基本的要求。

对于流服务，我们可以类比日常生活中的水流系统，比如水龙头和水池，这样会比较好理解。关于 hot shard 以及 Kinesis Data Stream scaling 的问题可以参考这篇文章：[Under the hood: Scaling your Kinesis data streams](https://aws.amazon.com/blogs/big-data/under-the-hood-scaling-your-kinesis-data-streams/)。

## Q045

`#s3`

`Once a month`, a company receives a `100 MB .csv file compressed with gzip`. The file contains 50,000 property listing records and is stored in `Amazon S3 Glacier`.
The company needs its data analyst to `query a subset of the data` for a specific vendor.
What is the `most cost-effective` solution?

A. Load the data into Amazon S3 and query it with Amazon S3 Select.

B. Query the data from Amazon S3 Glacier directly with Amazon Glacier Select.

C. Load the data to Amazon S3 and query it with Amazon Athena.

D. Load the data to Amazon S3 and query it with Amazon Redshift Spectrum.

### Answer - A

这道题也比较基础，考察如何查询 S3 Glacier 上的数据的子集。选项 B 不合理的原因是 Glacier Select 需要数据是未经压缩的、CSV 文件，题中的文件不符合要求。选项 A、C、D 都是可行的，但是 Athena 或者 Redshift Spectrum 会引入额外不必要的花费（它们都会按照扫描数据量、输出数据量以及查询数量来收费，S3 Select 只按返回数据量收费），同时效率也没有直接用 S3 Select 高。题目中的数据非常简单，每次接入都存放在同一个文件中，不需要做表的连接或者聚合操作，因此使用 S3 Select 就足够了。

## Q046

`#redshift`

A retail company is building its data warehouse solution using Amazon Redshift. As a part of that effort, the company is `loading hundreds of files into the fact table` created in its Amazon Redshift cluster. The company wants the solution to `achieve the highest throughput and optimally use cluster resources` when loading data into the company's fact table.
How should the company meet these requirements?

A. Use multiple COPY commands to load the data into the Amazon Redshift cluster.

B. Use S3DistCp to load multiple files into the Hadoop Distributed File System (HDFS) and use an HDFS connector to ingest the data into the Amazon Redshift cluster.

C. Use LOAD commands equal to the number of Amazon Redshift cluster nodes and load the data in parallel into each node.

D. Use a single COPY command to load the data into the Amazon Redshift cluster.

### Answer - D

这道题比较有意思，选项 A 和 D 是一对很容易混淆的方案。直觉来看，似乎调用多个 COPY 命令会让 Redshift 并行加载文件，其实不然。相反，COPY 命令本身就会自动并行运行，同时使用多个 COPY 命令反而会让数据加载变成串行的。因此排除选项 A。这是一个很容易记错的点。在 **[Q031]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q031)** 中有提到 COPY 命令对于文件的自动拆分。另外还可参考 [Use a single COPY command to load from multiple files](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-single-copy-command.html)。

对于选项 B，整个过程先将数据传到 HDFS，再从 HDFS 传到 Redshift，这明显增加了工作量。

对于选项 C，Redshift 并没有 LOAD 命令。

## Q047

`#athena` `#emr`

A data analyst is designing a solution to `interactively query datasets with SQL using a JDBC connection`. Users will `join data` stored in Amazon S3 in Apache ORC format with data stored in Amazon OpenSearch Service (Amazon Elasticsearch Service) and Amazon Aurora MySQL.
Which solution will provide the `MOST up-to-date` results?

A. Use AWS Glue jobs to ETL data from Amazon ES and Aurora MySQL to Amazon S3. Query the data with Amazon Athena.

B. Use Amazon DMS to stream data from Amazon ES and Aurora MySQL to Amazon Redshift. Query the data with Amazon Redshift.

C. Query all the datasets in place with Apache Spark SQL running on an AWS Glue developer endpoint.

D. Query all the datasets in place with Apache Presto running on Amazon EMR.

### Answer - D

从题目要求来看，需求是要交互式地查询多个数据源的数据，包括 S3、OpenSearch 和 Aurora MySQL，还需要结果是保持是最新的。那么最理想的方案就是有一个什么服务可以直接连到不同的数据源，直接运行查询，而不是把数据汇总到一个地方再查询。按照这种思路，虽然四个选项都是可行的，但选项 A、B 是需要将数据导出到同一个地方才行，因此暂且排除。

选项 C 的问题在于 Spark SQL 不能直接连到 OpenSearch，可能需要第三方的 connector，多少有些麻烦。

选项 D 是没有问题的，Presto 的功能本身就是同时处理多个数据源的数据。和选项 A、B 再对比一下，选项 D 实现起来简单，不需要经过数据迁移或者 ETL 的步骤，用到的数据自然是最新的。

这道题的答案意外地没有用到 AWS 的数据处理服务，而是依托 EMR 搭建了一个 Presto 服务。其实直接使用 AWS Athena 读取各数据源应该也是可以的，毕竟 Athena 的内核就是 Presto，又是 AWS 原生的服务，连接其他 AWS 服务没有问题。

## Q048

`#opensearch` `#lambda` `#glue`

A company developed a new elections reporting website that uses Amazon Kinesis Data Firehose to `deliver full logs` from AWS WAF to an Amazon S3 bucket.
The company is now seeking a `low-cost option` to perform this `infrequent data analysis` with `visualizations of logs` in a way that requires `minimal development effort`.
Which solution meets these requirements?

A. Use an AWS Glue crawler to create and update a table in the Glue data catalog from the logs. Use Athena to perform ad-hoc analyses and use Amazon QuickSight to develop data visualizations.

B. Create a second Kinesis Data Firehose delivery stream to deliver the log files to Amazon OpenSearch Service (Amazon Elasticsearch Service). Use Amazon ES to perform text-based searches of the logs for ad-hoc analyses and use OpenSearch Dashboards (Kibana) for data visualizations.

C. Create an AWS Lambda function to convert the logs into .csv format. Then add the function to the Kinesis Data Firehose transformation configuration. Use Amazon Redshift to perform ad-hoc analyses of the logs using SQL queries and use Amazon QuickSight to develop data visualizations.

D. Create an Amazon EMR cluster and use Amazon S3 as the data source. Create an Apache Spark job to perform ad-hoc analyses and use Amazon QuickSight to develop data visualizations.

### Answer - B?

这道题比较棘手，争议主要在选项 A 和 B 上。先说排除选项 C、D 的原因。主要都是因为引入了太多的外部服务。选项 C 用到了 Lambda 来转换文件格式，用 Redshift 做 ad-hoc 分析，最后用 QuickSight 可视化，这样会使资源和预算不足。选项 D 要用 EMR 来运行 Spark job 来进行分析，增加了开发的难度。

大多数人选择选项 A 的原因是便宜，由于题目中还提到 “infrequent data analysis”，会让人感觉运行的次数并不多，不经常发生。所以用 Glue crawler 来推算数据的 schema，存在 catalog 中，最后再用 Athena 和 QuickSight 来分析和可视化会比较节约。但其实这套流程下来，开发工作量并不小。

我更倾向于选项 B 的原因是，题中更重点的需求是 “minimal development effort”，开发的工作量要小。选项 B 就是一个很直截了当的方案，很好搭建流程。

## Q049

`#glue`

A large company has a `central data lake` to run analytics across different departments. Each department uses a `separate AWS account` and stores its data in an Amazon S3 bucket in that account. Each AWS account uses the AWS Glue Data Catalog as its data catalog. There are `different data lake access requirements based on roles`. Associate analysts should only have read access to their departmental data. Senior data analysts can have access in multiple departments including theirs, but for a `subset of columns only`.
Which solution achieves these required access patterns to minimize costs and administrative tasks?

A. Consolidate all AWS accounts into one account. Create different S3 buckets for each department and move all the data from every account to the central data lake account. Migrate the individual data catalogs into a central data catalog and apply fine-grained permissions to give to each user the required access to tables and databases in AWS Glue and Amazon S3.

B. Keep the account structure and the individual AWS Glue catalogs on each account. Add a central data lake account and use AWS Glue to catalog data from various accounts. Configure cross-account access for AWS Glue crawlers to scan the data in each departmental S3 bucket to identify the schema and populate the catalog. Add the senior data analysts into the central account and apply highly detailed access controls in the Data Catalog and Amazon S3.

C. Set up an individual AWS account for the central data lake. Use AWS Lake Formation to catalog the cross-account locations. On each individual S3 bucket, modify the bucket policy to grant S3 permissions to the Lake Formation service-linked role. Use Lake Formation permissions to add fine-grained access controls to allow senior analysts to view specific tables and columns.

D. Set up an individual AWS account for the central data lake and configure a central S3 bucket. Use an AWS Lake Formation blueprint to move the data from the various buckets into the central S3 bucket. On each individual bucket, modify the bucket policy to grant S3 permissions to the Lake Formation service-linked role. Use Lake Formation permissions to add fine-grained access controls for both associate and senior analysts to view specific tables and columns.

### Answer - B

题目中场景描述的是，一个中央数据湖需要分析来自多个部门的数据，每个部门有自己单独的 AWS 账户以及 Glue Catalog，还要为不同的分析人员分配不同的访问策略。

一个理想的方案应该是这个数据湖拥有所有部门数据的元数据（即数据湖本身就是所有部门数据的 Catalog），这样在做分析的时候就不需要移动数据；同时，为了访问控制，高级分析人员能够访问到这个中央数据 Catalog，其他分析人员只要保留现有的自己部门数据的访问权限即可。

而麻烦的方案则是，将所有部门的数据移动到同一个账户下再进行 Catalog，或者对每一个部门账户都修改访问设置。前者浪费资金，后者工作量大，浪费时间且易出错。

由以上分析可排除选项 A、D。选项 B、C 之间争议比较多，我更倾向于选择 B。因为选项 B、C 都是要为中央数据湖创建一个单独的账户，但选项 C 多出了一个 Lake Formation 服务，同时还需要逐个修改所有部门的 S3 bucket policy，这无疑会增加很多工作量及支出。而且题目并不是要新建一个数据湖，如果是从头新建数据湖的话，Lake Formation 可能会简单一些。


## Q050

`#kinesis-data-streams` `#kinesis-data-firehose` `#kinesis-analytics` `#opensearch`

A company wants to improve user satisfaction for its smart home system by adding more features to its recommendation engine. Each sensor asynchronously pushes its `nested JSON data` into Amazon Kinesis Data Streams using the Kinesis Producer Library (KPL) in Java. Statistics from a set of failed sensors showed that, `when a sensor is malfunctioning, its recorded data is not always sent to the cloud`.
The company needs a solution that offers `near-real-time analytics` on the data from the most updated sensors.
Which solution enables the company to meet these requirements?

A. Set the RecordMaxBufferedTime property of the KPL to "0" to disable the buffering on the sensor side. Use Kinesis Data Analytics to enrich the data based on a company-developed anomaly detection SQL script. Push the enriched data to a fleet of Kinesis data streams and enable the data transformation feature to flatten the JSON file. Instantiate a dense storage Amazon Redshift cluster and use it as the destination for the Kinesis Data Firehose delivery stream.

B. Update the sensors code to use the PutRecord/PutRecords call from the Kinesis Data Streams API with the AWS SDK for Java. Use Kinesis Data Analytics to enrich the data based on a company-developed anomaly detection SQL script. Direct the output of KDA application to a Kinesis Data Firehose delivery stream, enable the data transformation feature to flatten the JSON file, and set the Kinesis Data Firehose destination to an Amazon OpenSearch Service (Amazon Elasticsearch Service) cluster.

C. Set the RecordMaxBufferedTime property of the KPL to "0" to disable the buffering on the sensor side. Connect for each stream a dedicated Kinesis Data Firehose delivery stream and enable the data transformation feature to flatten the JSON file before sending it to an Amazon S3 bucket. Load the S3 data into an Amazon Redshift cluster.

D. Update the sensors code to use the PutRecord/PutRecords call from the Kinesis Data Streams API with the AWS SDK for Java. Use AWS Glue to fetch and process data from the stream using the Kinesis Client Library (KCL). Instantiate an Amazon Elasticsearch Service cluster and use AWS Lambda to directly push data into it.

### Answer - B

这道题选项比题目复杂。题中条件很简单，传感器通过 KPL 将其数据以异步的方式传到 Kinesis Data Streams，如果传感器出故障可能会漏传数据。现在需要对最新的数据进行近实时地分析。

首先要分析一下传感器用异步的方式传输数据会发生什么。异步指的是，传感器持续地收集数据，并选恰当时机向 KDS 发送该数据，但并不需要等数据传输完成，只要完成发送动作，传感器就认为发送完成。这样一来，如果传感器和 KDS 之间的链路上有问题，导致数据传输失败，传感器是无从知晓的；同时 KDS 端也会丢失这一次的数据，导致系统所分析的数据不是最新的。

因此要解决这个问题，首先是要保证 KDS 上接收的都是最新数据。这就要求传感器不能再以异步的方式发送数据了，要改成同步的。选项中分了两种方案：一是将 KPL 的 `RecordMaxBufferedTime` 参数设成 0，目的是让 KPL 不再缓存，数据就绪就发出去；二是使用 AWS SDK 里的 `PutRecord/PutRecords` 方法，代替 KPL。前者并没有解决异步发送的问题，而后者是同步的，因此应该用第二种方式。排除选项 A、C。

接下来就是如何近实时地分析数据。其实在 DAS-C01 的考题中，目前遇到的所有提到 “近实时” 的需求，大概率都是和 Kinesis Firehose 或者 OpenSearch 有关的。选项 D 中的 Glue 更适用于批处理。因此选择 B。

另外再补充几点其他选项的问题：
1. 选项 A，KDS 没有 transformation 的处理功能，除非连入 Lambda。
2. 选项 D，从 SDK 传入 KDS 的数据不能被 KCL 消费。
