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
