---
layout: post
title: "AWS Data Analytics Specialty (DAS-C01) 认证例题整理（二）"
description: "这篇文章里包含了笔者整理的关于 DAS-C01 认证的样题，共 50 道题。这些题目都是从网站、论坛上搜集来的。答案是笔者通过查阅资料并结合他人解题思路后得出的，不能保证正确性。读者需认真思考，仔细辨别，欢迎提出不同意见。"
tags: [aws, das-c01, reference]
---

## Q051

`#quicksight` `#visual-type`

A global company has different sub-organizations, and each sub-organization sells its products and services in various countries. The company's senior leadership wants to quickly identify which sub-organization is the strongest performer in each country. `All sales data is stored in Amazon S3 in Parquet format`.
Which approach can `provide the visuals` that senior leadership requested with the `least amount of effort`?  

A. Use Amazon QuickSight with Amazon Athena as the data source. Use heat maps as the visual type.

B. Use Amazon QuickSight with Amazon S3 as the data source. Use heat maps as the visual type.

C. Use Amazon QuickSight with Amazon Athena as the data source. Use pivot tables as the visual type.

D. Use Amazon QuickSight with Amazon S3 as the data source. Use pivot tables as the visual type.

### Answer - A

这道题考察 QuickSight 的用法。根据题中条件，各子机构的数据是以 Parquet 格式的文件存储在 S3 bucket 中的。如果想用 QuickSight 来展示数据分布，则需要用 Athena 来处理数据，再用 QuickSight 为这些数据建立  dataset。不用 S3 作为 dataset 的原因有二：
1. QuickSight 不支持访问 S3 上的 Parquet 数据，QuickSight 只能使用 csv、elf 等文件，同时还需要在 S3 bucket 里添加一个 manifest 文件。
2. 使用 S3 的数据时，QuickSight 会自动将它们导入到 SPICE 中，也就是要额外付出 QuickSight 的存储和运算资源以及开发成本。

再说使用哪一种图表来表示数据分析的结果。因为要求的是最佳的子机构，用热图可以比用透视表更直观。

## Q052

`#opensearch` `#glue` `#redshift` `#athena` `#s3`

A company has `1 million scanned documents stored as image files` in Amazon S3. The documents contain typewritten application forms with information including the applicant first name, applicant last name, application date, application type, and application text. The company has developed a `machine learning algorithm to extract the metadata values` from the scanned documents. The company wants to allow internal data analysts to analyze and `find applications` using the applicant name, application date, or `application text`. `The original images should also be downloadable`. `Cost control is secondary to query performance`.
Which solution organizes the images and metadata to drive insights while meeting the requirements?

A. For each image, use object tags to add the metadata. Use Amazon S3 Select to retrieve the files based on the applicant name and application date.

B. Index the metadata and the Amazon S3 location of the image file in Amazon OpenSearch Service (Amazon Elasticsearch Service). Allow the data analysts to use OpenSearch Dashboards (Kibana) to submit queries to the Amazon OpenSearch Service (Amazon Elasticsearch Service) cluster.

C. Store the metadata and the Amazon S3 location of the image file in an Amazon Redshift table. Allow the data analysts to run ad-hoc queries on the table.

D. Store the metadata and the Amazon S3 location of the image files in an Apache Parquet file in Amazon S3, and define a table in the AWS Glue Data Catalog. Allow data analysts to use Amazon Athena to submit custom queries.

### Answer - B

这道题的关键在于要求能够通过 application text 来查找 application，也就是文本搜索。几个选项中只有 OpenSearch 能做到。

选项 A 中使用的 object tag 不能作为搜索条件，即使将那些 metadata 标记在了 tag 中，分析员也没办法搜索到。

选项 C 中提到用 Redshift 来存储 metadata 以及那些文件在 S3 中的位置，但 Redshift 本身是一个数据仓库的解决方案，更适用于结构化的数据，而且也不推荐存储二进制文件。对于图像文件来说，Redshift 不是一个好的方案。

选项 D 中的 Glue table 确实是一个可行的方案，最开始看题的时候，第一反应就是 Glue。但它依然不适用于文本搜索，即使 Glue table 中有一列专门存储 application text，但是在 Athena 查询的时候性能会很差。

## Q053

`#kinesis-data-streams`

A mobile gaming company wants to capture data from its gaming app and make the data available `for analysis immediately`. The data record size will be `approximately 20 KB`. The company is concerned about `achieving optimal throughput from each device`. Additionally, the company wants to `develop a data stream processing application with dedicated throughput for each consumer`.
Which solution would achieve this goal?

A. Have the app call the PutRecords API to send data to Amazon Kinesis Data Streams. Use the enhanced fan-out feature while consuming the data.

B. Have the app call the PutRecordBatch API to send data to Amazon Kinesis Data Firehose. Submit a support case to enable dedicated throughput on the account.

C. Have the app use Amazon Kinesis Producer Library (KPL) to send data to Kinesis Data Firehose. Use the enhanced fan-out feature while consuming the data.

D. Have the app call the PutRecords API to send data to Amazon Kinesis Data Streams. Host the stream-processing application on Amazon EC2 with Auto Scaling.

### Answer - A

这道题是一个标准的考察 kinesis data streams 的题目，场景比较典型。数据大小比较小，需要即刻进行分析，消费者还需要获得专有的吞吐量。

首先看排除选项 B、C 的原因，kinesis data firehose 是近实时的流数据迁移方案，最低的延时也要 60 秒，不满足立刻分析的需求。

排除选项 D 的原因是 EC2 的自动扩展配置只能允许在集群资源不够的情况下，扩充 EC2 的节点个数，但没有办法让运行其上的流处理程序具有专门的吞吐量。

Kinesis Data Stream 中的数据在消费时，每个 shard 默认是分配 2 MB/s 的吞吐量，也就是说不管有多少的消费者来消费数据，它们都只能共享这 2 MB/s 的带宽。要想为每个消费者都提供 2 MB/s 的带宽，就必须使用 enhanced fan-out 特性。因此选 A。

另外再补充几点关于 KPL 和 PutRecords API 的内容。KPL 的特点是有延迟，因为它要将待发送的数据进行缓存，缓存的目的是为了尽可能的将传输数据的大小顶到上限（1 MB/s/shard）。它采用了两个步骤：1. 聚合（aggregation）：将多条小记录合并成一条大记录；2. 集中（collection）：将这些大记录通过一次 PutRecords API 调用分别发送到不同的 shard 中。相关的解释在 **[Q050]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q050)** 中也提到过。所以我个人觉得，如果题目中有一个使用 “KPL + enhanced fan-out” 的选项，也许这个方案会更好地利用带宽。至于延迟是可以配置的，默认的 `RecordMaxBufferedTime` 参数值是 100ms，可以设置得更低一些以达到更快发送的目的。

## Q054

`#redshift` `#redshift-spectrum` `#quicksight` `#emr`

A marketing company wants to improve its reporting and business intelligence capabilities. During the planning phase, the company interviewed the relevant stakeholders and discovered that:
✑ `The operations team reports are run hourly for the current month's data`.
✑ The sales team wants to `use multiple Amazon QuickSight dashboards` to show a `rolling view of the last 30 days` based on several categories. The sales team also wants to `view the data as soon` as it reaches the reporting backend.
✑ `The finance team's reports are run daily for last month's data and once a month for the last 24 months of data`.
Currently, there is `400 TB of data` in the system with an expected additional `100 TB added every month`. The company is looking for a solution that is `as cost-effective as possible`.
Which solution meets the company's requirements?

A. Store the last 24 months of data in Amazon Redshift. Configure Amazon QuickSight with Amazon Redshift as the data source.

B. Store the last 2 months of data in Amazon Redshift and the rest of the months in Amazon S3. Set up an external schema and table for Amazon Redshift Spectrum. Configure Amazon QuickSight with Amazon Redshift as the data source.

C. Store the last 24 months of data in Amazon S3 and query it using Amazon Redshift Spectrum. Configure Amazon QuickSight with Amazon Redshift Spectrum as the data source.

D. Store the last 2 months of data in Amazon Redshift and the rest of the months in Amazon S3. Use a long-running Amazon EMR with Apache Spark cluster to query the data as needed. Configure Amazon QuickSight with Amazon EMR as the data source.

### Answer - B

这是一道基础题。考察 Redshift 和 Redshift Spectrum 的使用场景。按题中描述，频繁访问的数据最早到两个月前，历史数据却要追溯到两年前，而且每个月都会新进入一大批数据。这就说明需要将频繁访问的数据和不频繁访问的数据分隔开。短期数据需要更高效的处理方案，即 Redshift 集群；历史数据需要更便宜的存储方案，即 S3。它们之间联合的查询操作可以通过 Redshift Spectrum 来实现。由此可排除选项 A、C。

排除选项 D 的原因是这里使用 EMR 就浪费了 Redshift 的资源，Redshift 原本就是为大数据数仓而设计的，很适合 TB 级甚至 PB 级数据的查询。EMR 在这里做的无非也就是这些。一般来说，对于 “cost-effective” 或者 “minimal development effort” 这样的需求，如果 AWS 已经有了现成的解决方案，就尽量不选择 EMR。

再补充一点，选项 C 中提到把所有的数据都存入 S3，然后利用 Redshift Spectrum 做查询。对于这么大的数据量以及如此频繁的查询来说，就意味着每小时都需要对几百 TB 的数据进行一次扫描，这个花销是很大的。这也是要把频繁访问与偶尔访问的数据分隔开的原因。关于 Redshift Spectrum 的收费以及最佳实践可以参考这篇文档：[How do I calculate the query charges in Amazon Redshift Spectrum?](https://repost.aws/knowledge-center/redshift-spectrum-query-charges)

## Q055

`#glue` `#s3` `#athena` `#redshift` `#emr`

A media company wants to perform machine learning and analytics on the data residing in its `Amazon S3 data lake`. There are two data transformation requirements that will enable the consumers within the company to create reports:
✑ `Daily transformations of 300 GB of data` with `different file formats landing` in Amazon S3 at a scheduled time.
✑ `One-time transformations of terabytes of archived data` residing in the S3 data lake.
Which combination of solutions `cost-effectively` meets the company's requirements for transforming the data? (Choose three.)

A. For daily incoming data, use AWS Glue crawlers to scan and identify the schema.

B. For daily incoming data, use Amazon Athena to scan and identify the schema.

C. For daily incoming data, use Amazon Redshift to perform transformations.

D. For daily incoming data, use AWS Glue workflows with AWS Glue jobs to perform transformations.

E. For archived data, use Amazon EMR to perform data transformations.

F. For archived data, use Amazon SageMaker to perform data transformations.

### Answer - ADE

这道题比较简单，考察数据 ETL 时选用什么服务。每天都有 300 GB 的不同格式的文件需要进入数据湖，那么文件的 schema 最好是能自动识别，所以要用 Glue crawlers，而 Athena 不能识别 schema，因此排除选项 B。

识别了 schema 之后需要将数据传入 S3，Redshift 是一个数仓工具，不适合做数据的传输，适合做传输的是 Glue workflow，利用 glue job 来进行调度和传输。因此排除选项 C。

对于历史数据来说，使用 EMR 比 SageMaker 更为合适，因为后者是一个机器学习的平台，它适合对数据进行模型的构建、学习和部署，但不适用于数据的传输。而 EMR 上可以搭建各种适用的传输工具，因此排除选项 F。

## Q056

`#kinesis-data-streams` `#kinesis-data-firehose` `#lambda` `#s3`

A hospital uses wearable medical sensor devices to collect data from patients. The hospital is architecting a `near-real-time solution` that can `ingest the data securely at scale`. The solution should also be able to `remove the patient's protected health information (PHI)` from the streaming data and store the data in `durable storage`.
Which solution meets these requirements with the `least operational overhead`?

A. Ingest the data using Amazon Kinesis Data Streams, which invokes an AWS Lambda function using Kinesis Client Library (KCL) to remove all PHI. Write the data in Amazon S3.

B. Ingest the data using Amazon Kinesis Data Firehose to write the data to Amazon S3. Have Amazon S3 trigger an AWS Lambda function that parses the sensor data to remove all PHI in Amazon S3.

C. Ingest the data using Amazon Kinesis Data Streams to write the data to Amazon S3. Have the data stream launch an AWS Lambda function that parses the sensor data and removes all PHI in Amazon S3.

D. Ingest the data using Amazon Kinesis Data Firehose to write the data to Amazon S3. Implement a transformation AWS Lambda function that parses the sensor data to remove all PHI.

### Answer - D

题目要求近实时地将数据导入一个持久存储的服务中，同时还要删去敏感信息。从之前的那些题目中来看，一般提到 “近实时” 大概率是和 kinesis data firehose 有关的。题目中的选项里涉及到了 kinesis data streams 和 kinesis data firehose，我们可以辨别一下是否能排除 kinesis data streams 的选项。

在题中描述的数据接入的场景中，我们应该能分析出一共需要以下三个步骤：

1. 用某个工具接到数据
2. 对数据进行处理，删除敏感信息
3. 将数据存入 S3（持久存储）

步骤二要在步骤三之前的原因是，这样避免了数据已经落盘再进行处理所造成的资源（主要是数据读写工作）的浪费，而且也是为了数据安全，敏感信息应该尽早去除，不要存放在持久存储服务中。由此可以排除选项 B、C。

选项 A 也是一种可行的办法，但和选项 D 相比，它会更复杂一些。因为如果使用 KCL 来处理数据，还需用 DynamoDB 记录处理进度，所以需要额外的支出，并且要为 DynamoDB 分配资源，这会增加运维的开销。因此排除选项 A。

## Q057

`#emr` `#security`

A company is `migrating its existing on-premises ETL jobs to Amazon EMR`. The code consists of a series of jobs written in Java. The company needs to `reduce overhead` for the system administrators `without changing the underlying code`. Due to the sensitivity of the data, compliance requires that the company `use root device volume encryption on all nodes` in the cluster. Corporate standards require that `environments be provisioned through AWS CloudFormation` when possible.
Which solution satisfies these requirements?

A. Install open-source Hadoop on Amazon EC2 instances with encrypted root device volumes. Configure the cluster in the CloudFormation template.

B. Use a CloudFormation template to launch an EMR cluster. In the configuration section of the cluster, define a bootstrap action to enable TLS.

C. Create a custom AMI with encrypted root device volumes. Configure Amazon EMR to use the custom AMI using the CustomAmild property in the CloudFormation template.

D. Use a CloudFormation template to launch an EMR cluster. In the configuration section of the cluster, define a bootstrap action to encrypt the root device volume of every node.

### Answer - C

按题目要求，需要对每个节点都进行 root volume encryption，同时还要用 CloudFormation 来搭建 EMR 的集群环境。所以用户应该先自定义一个对 root volume 加密的 AMI，然后再令 CloudFormation 使用该 AMI 来创建 EMR 集群。

选项 A 要使用开源的 Hadoop，CloudFormation 不支持这个选项。另外这个方式也会涉及到很多的运维开销。

选项 B 提到启用 TLS，但并没有说如何加密 root volume。

选项 D 提到使用 bootstrap action 对 root volume 加密，理论上是可行的，但是需要的配置、运维花销会很大。

关于题中的 root volume encryption 场景，官方文档中有提到解决方案：[Using a custom AMI](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-custom-ami.html)。还有一篇文章供参考：[How do I set the properties of a root volume for an Amazon EC2 instance that I created using an AWS CloudFormation template?](https://repost.aws/knowledge-center/cloudformation-root-volume-property)。

## Q058

`#s3` `#redshift` `#athena`

A transportation company uses IoT sensors attached to trucks to collect vehicle data for its global delivery fleet. The company currently sends the sensor data in `small .csv files to Amazon S3`. The files are then loaded into a `10-node Amazon Redshift cluster` with `two slices per node` and queried using both Amazon Athena and Amazon Redshift. The company wants to `optimize the files` to `reduce the cost of querying` and also `improve the speed of data loading` into the Amazon Redshift cluster.
Which solution meets these requirements?

A. Use AWS Glue to convert all the files from .csv to a single large Apache Parquet file. COPY the file into Amazon Redshift and query the file with Athena from Amazon S3.

B. Use Amazon EMR to convert each .csv file to Apache Avro. COPY the files into Amazon Redshift and query the file with Athena from Amazon S3.

C. Use AWS Glue to convert the files from .csv to a single large Apache ORC file. COPY the file into Amazon Redshift and query the file with Athena from Amazon S3.

D. Use AWS Glue to convert the files from .csv to Apache Parquet to create 20 Parquet files. COPY the files into Amazon Redshift and query the files with Athena from Amazon S3.

### Answer - D

类似的关于 LOAD 数据到 Redshift 中的题目，已经见过很多次了（见 [Q019]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q019)、[Q031]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q031)、[Q046]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q046)）。重点就是要利用 Redshift 的 MPP，并行地加载数据。同时还要利用列式存储的优势，将文件转为列式存储的格式（Avro 是行式存储）。

## Q059

`#kinesis-data-firehose` `#kinesis-data-streams` `#opensearch`

An online retail company with millions of users around the globe wants to improve its e-commerce analytics capabilities. Currently, clickstream data is uploaded directly `to Amazon S3 as compressed files`. `Several times each day`, an application running on Amazon EC2 processes the data and makes search options and reports available for visualization by editors and marketers. The company wants to make website clicks and aggregated data `available to editors and marketers in minutes` to enable them to connect with users more effectively.
Which options will help meet these requirements in the MOST efficient way？ (Choose two.)

A. Use Amazon Kinesis Data Firehose to upload compressed and batched clickstream records to Amazon OpenSearch Service (Amazon Elasticsearch Service).

B. Upload clickstream records to Amazon S3 as compressed files. Then use AWS Lambda to send data to Amazon OpenSearch Service (Amazon Elasticsearch Service) from Amazon S3.

C. Use Amazon OpenSearch Service (Amazon Elasticsearch Service) deployed on Amazon EC2 to aggregate, filter, and process the data. Refresh content performance dashboards in near-real time.

D. Use OpenSearch Dashboards (Kibana) to aggregate, filter, and visualize the data stored in Amazon OpenSearch Service (Amazon Elasticsearch Service). Refresh content performance dashboards in near-real time.

E. Upload clickstream records from Amazon S3 to Amazon Kinesis Data Streams and use a Kinesis Data Streams consumer to send records to Amazon OpenSearch Service (Amazon Elasticsearch Service).

### Answer - AD

题目中提到了 “available in minutes” 其实就暗示了这是一个近实时地场景，在这个场景下，用 kinesis data firehose 将数据接入 OpenSearch，再用 OpenSearch 进行分析、查找以及可视化是最高效的解决方案。

其他两个关于数据接入的选项中，选项 B 更适合做一个实时的方案，选项 E 中 kinesis data streams 不能直接将数据输出到 OpenSearch 中。

## Q060

`#kinesis-data-streams`

A company is streaming its high-volume billing data (`100 MBps`) to Amazon Kinesis Data Streams. A data analyst `partitioned the data on account_id` to ensure that all records belonging to an account go to the same Kinesis shard and order is maintained. While building a custom consumer using the Kinesis Java SDK, the data analyst notices that, sometimes, `the messages arrive out of order for account_id`. Upon further investigation, the data analyst discovers `the messages that are out of order seem to be arriving from different shards for the same account_id and are seen when a stream resize runs`.
What is an explanation for this behavior and what is the solution?

A. There are multiple shards in a stream and order needs to be maintained in the shard. The data analyst needs to make sure there is only a single shard in the stream and no stream resize runs.

B. The hash key generation process for the records is not working correctly. The data analyst should generate an explicit hash key on the producer side so the records are directed to the appropriate shard accurately.

C. The records are not being received by Kinesis Data Streams in order. The producer should use the PutRecords API call instead of the PutRecord API call with the SequenceNumberForOrdering parameter.

D. The consumer is not processing the parent shard completely before processing the child shards after a stream resize. The data analyst should process the parent shard completely first before processing the child shards.

### Answer - D

这道题看起来比较复杂，但实际上描述的场景很典型，就是由于 kinesis data streams 进行了 resize，导致数据流中的数据乱序了。根本原因就如选项 D 所说，在 resize 的时候，parent shard 里的数据尚未完全消费就开始消费 child shards 中的数据了。

其他几个选项的原因分析得都不对。选项 A 说应该只保留一个 shard，这样顺序是保证了，但是效率却降下来了。选项 B 说是 hash 函数运行得有问题，hash 函数是用在给 account_id 分区的时候，相同的 account_id 会有相同的 hash 值，这个一般是不会出错的，而且更不会总是出错。选项 C 说 KDS 接到数据的时候就是乱序的，如果是这样的话，乱序就不会只发生在 resize 运行的时候。

另外，KCL 自带 resize 的处理逻辑，这里如果弃用 SDK 而改用 KCL 就会解决这个问题。用 SDK 来实现 resize 的处理会有些复杂。
