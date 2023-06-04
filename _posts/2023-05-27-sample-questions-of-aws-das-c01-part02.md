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

类似的关于 LOAD 数据到 Redshift 中的题目，已经见过很多次了（见 [Q019]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q019)、[Q031]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q031)、[Q035]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q035)、[Q046]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q046)）。重点就是要利用 Redshift 的 MPP，并行地加载数据。同时还要利用列式存储的优势，将文件转为列式存储的格式（Avro 是行式存储）。

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

## Q061

`#kinesis-data-streams` `#lambda` `#opensearch` `#glue`

A media analytics company consumes a stream of social media posts. The posts are sent to an Amazon Kinesis data stream `partitioned on user_id`. An AWS  
`Lambda function retrieves the records and validates the content` before loading the posts into an Amazon OpenSearch Service (Amazon Elasticsearch Service) cluster. The validation process `needs to receive the posts for a given user in the order` they were received by the Kinesis data stream.
During peak hours, the social media posts take more than an hour to appear in the Amazon OpenSearch Service (Amazon ES) cluster. A data analytics specialist must implement a solution that `reduces this latency` with the `least possible operational overhead`.
Which solution meets these requirements?

A. Migrate the validation process from Lambda to AWS Glue.

B. Migrate the Lambda consumers from standard data stream iterators to an HTTP/2 stream consumer.

C. Increase the number of shards in the Kinesis data stream.

D. Send the posts stream to Amazon Managed Streaming for Apache Kafka instead of the Kinesis data stream.

### Answer - C

分析题目场景可知，出现性能问题的原因是在高峰时段数据量大导致数据处理不过来。那么为了提高数据处理性能，最简单直接的方式就是增加处理节点，并行运算。题中的数据处理架构如下图所示：

![das-c01-q061-data-processing-architecture](https://zhenqi-imagebed.s3.ap-east-1.amazonaws.com/uploaded_date=2023-06/das-c01-q061-bae772baaed79bd54532df1f9538362a.jpg)

每次 Lambda 调用都可以处理来自一个 Kinesis shard 的数据。因此如果增加 shard 数量的话，就会并行地调用多个 Lambda 方法来处理多组数据，由此提升数据处理性能（如图中虚线部分所示）。

那么其他几个选项有什么问题呢？选项 A 用到 Glue，它背后的运行环境是 Spark，确实有可能达到比 Lambda 更快的运算速度，但是开发难度也会增加，在有更简单方案的时候可以不选它。选项 D 主要是更换了数据管道的服务，增加了运维以及开发的难度，但并不能保证效率的提升。选项 B 也类似，HTTP/2 的方式其实是用到了 Kinesis Data Streams 的 enhanced fan-out 特性，提升了数据传输速度（带宽），看起来似乎可以解决高峰时期数据量大的问题。但这个方案并没有抓住性能瓶颈的根因，只提升数据传输速度不能解决问题。

## Q062

`#kinesis-data-streams` `#optimization`

A company launched a service that `produces millions of messages every day` and uses Amazon Kinesis Data Streams as the streaming service.
The company `uses the Kinesis SDK to write data` to Kinesis Data Streams. A few months after launch, a data analyst found that `write performance is significantly reduced`. The data analyst investigated the metrics and determined that `Kinesis is throttling the write requests`. The data analyst wants to address this issue `without significant changes to the architecture`.
Which actions should the data analyst take to resolve this issue? (Choose two.)  

A. Increase the Kinesis Data Streams retention period to reduce throttling.

B. Replace the Kinesis API-based data ingestion mechanism with Kinesis Agent.

C. Increase the number of shards in the stream using the UpdateShardCount API.

D. Choose partition keys in a way that results in a uniform record distribution across shards.

E. Customize the application code to include retry logic to improve performance.

### Answer - CD

按题目中，服务上线几个月后才出现写数据性能降低，再考虑到 Kinesis SDK 写数据时相同 partition key 会进入同一个 shard，可以想到性能问题有可能是因为数据量太多，现有 shard 数量太少，因此阻碍了数据写入，也可能是出现了 hot shard。因此可以尝试增加 shard 数量，并且让 partition key 均匀分布以解决问题，即选择选项 C、D。

选项 A、E 都只是增加数据在 Kinesis Stream 中的存活时间，但均不能解决数据太多或是 hot shard 的问题，因此这两种方案意义不大。选项 B 是用 Kinesis Agent 来代替 SDK，同样不能解决这两个问题。

## Q063

`#kinesis-data-streams` `#emr` `#glue` `#kinesis-data-firehose` `#lambda`

A smart home automation company must efficiently ingest and process messages from various connected devices and sensors. The majority of these messages are `comprised of a large number of small files`. These messages are ingested using Amazon Kinesis Data Streams and sent to Amazon S3 using a Kinesis data stream consumer application. The Amazon S3 message data is then passed through `a processing pipeline built on Amazon EMR running scheduled PySpark jobs`.
The data platform team manages data processing and is concerned about the `efficiency and cost of downstream data processing`. They want to continue to use  
PySpark.
Which solution improves the efficiency of the data processing jobs and is well architected?

A. Send the sensor and devices data directly to a Kinesis Data Firehose delivery stream to send the data to Amazon S3 with Apache Parquet record format conversion enabled. Use Amazon EMR running PySpark to process the data in Amazon S3.

B. Set up an AWS Lambda function with a Python runtime environment. Process individual Kinesis data stream messages from the connected devices and sensors using Lambda.

C. Launch an Amazon Redshift cluster. Copy the collected data from Amazon S3 to Amazon Redshift and move the data processing jobs from Amazon EMR to Amazon Redshift.

D. Set up AWS Glue Python jobs to merge the small data files in Amazon S3 into larger files and transform them to Apache Parquet format. Migrate the downstream PySpark jobs from Amazon EMR to AWS Glue.

### Answer - D

通过题目要求 “需要继续使用 PySpark” 可以排除选项 C，因为 Redshift 不能运行 PySpark 程序。选项 B 提到用 Lambda 运行 Python 脚本来处理每一条来自 Kinesis Data Streams 的数据，这个方案改变了整个系统的运行方式，从定时调度的批处理变成了持续运行的流处理。题目中没有相关要求，同时这种处理方式既不经济，又没有明显地提示说比批处理强，因此排除选项 B。

这道题的争议主要是选项 A、D 之间，它们都是可实现的。选项 A 采用 Kinesis Data Firehose 代替了 Kinesis Data Streams 来接入数据，在接入的过程中还能顺便将文件转为 Parquet 格式。但笔者认为它的问题在于没有减少 EMR 所造成的花销，另外对于小文件也没有处理，顶多是靠 Firehose 自身的延时来合并若干记录为一个文件。

而选项 D 用 Glue 来代替了 EMR，更好地运用了其自身的运算资源以及调度系统，保留了 PySpark 的使用，减少了开销，同时还合并了大量小文件，为下游的处理做好了准备。这个选项笔者认为更适合题中场景。

## Q064

`#s3` `#redshift`

A large financial company is running its ETL process. Part of this process is to `move data from Amazon S3 into an Amazon Redshift cluster`. The company wants to use the `most cost-efficient` method to load the dataset into Amazon Redshift.
Which combination of steps would meet these requirements? (Choose two.)

A. Use the COPY command with the manifest file to load data into Amazon Redshift.

B. Use S3DistCp to load files into Amazon Redshift.

C. Use temporary staging tables during the loading process.

D. Use the UNLOAD command to upload data into Amazon Redshift.

E. Use Amazon Redshift Spectrum to query files from Amazon S3.

### Answer - AC

又是一道考察 Redshift COPY 数据的题目，之前已经在  [Q019]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q019)、[Q031]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q031)、[Q035]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q035)、[Q046]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q046) 以及 [Q058]({{ site.baseurl }}{% link _posts/2023-05-27-sample-questions-of-aws-das-c01-part02.md %}#q058) 中见到过了，选项 A 肯定是正确答案之一。选项 C 提到在载入数据的过程中使用用临时的 staging table，它的好处是帮助执行 upsert 操作，可参考 [Q006]({{ site.baseurl }}{% link _posts/2023-04-13-sample-questions-of-aws-das-c01-part01.md %}#q006) 中的解释。因此这道题的答案是选项 A、C。

选项 B 主要是用于 S3 和 HDFS 之间的文件传输。选项 D 是将数据从 Redshift 到处到 S3 的。选项 E 并不涉及数据传输，不符合题目要求。
