---
layout: post
title: "在 AWS ECS 上部署 Docker 容器"
description: "本文介绍了在 AWS ECS 服务上部署 Docker 容器的方法，通过阅读本文，读者可以了解到基本的操作流程以及一些注意事项"
tags: [aws, ecs, docker, how-to-guide]
---

# 准备工作

## AWS 准备

首先，需要创建 AWS 账号，并且拥有一个具有 ECS 权限的用户。

其次，需要准备好 VPC、子网、安全组的设置。

以上操作，可以参考 AWS 文档：[Set up to use Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html). 如果之后用到更为复杂的网络配置，可以根据需求具体分析。

## 代码准备

### Container IP

在 docker 的机制中，它会给同一个 docker-compose 文件内的服务分配同一个内部网络，服务之间的通信只需将地址设定为服务名称即可。当我们使用 ECS 来运行容器时，由同一个 Task definition 配置的 container，都可以用本地地址 `localhost` 来指代。即多个 container 都是属于同一个 localhost 的，他们之间的通信是由 localhost + 端口号的方式定位的。因此如果容器 A 的接口是 80，那么其他要连接到容器 A 的容器就需要访问 localhost:80。

### 环境变量

通过上面的描述可以看出，本地运行 container 和在 ECS 上运行 container 会有些许不同的差别。这些不同可以通过环境变量来动态设置。如果容器 A 需要访问 `container_b:port`，那么可以将代码修改为 `${ENV.CONTAINER_B_URL}:{ENV.PORT}`，然后在 Task definition 中设置对应的环境变量。

# 定义任务

进入 Elastic Container Service 控制台。关于 ECS 的组件及基本概念可以参考[Amazon ECS Components](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/welcome-features.html)。

在控制台内，首先创建一个集群（Cluster），集群的配置中，主要需要定义名称、选择 VPC 和子网，定义基础设施（Infrastructure）—— 即运行环境的资源是用 [Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html) 还是 [EC2](https://docs.aws.amazon.com/ec2/?icmpid=docs_homepage_compute) 来分配，是否需要用 [Container Insights](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-container-insights.html) 来监控集群运行状态，最后可选择是否要添加标签。

创建集群后，需要添加 [Task Definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)，它的作用相当于是一个 `docker-compose.yaml` 文件，用于编排容器。其中需要指定 Task definition family 的名字，添加容器的配置：镜像名、镜像 URI、端口映射、环境变量、健康检测以及 Docker 的配置（entrypoint、command、工作目录），设置适用的运行环境（CPU、内存、运行 container 所使用的角色），配置存储空间及监测方式，最后可选是否要添加标签。有了 Task definition 就有了运行容器的蓝图。

使用控制台来定义任务时，有很多选项都不能设置，例如容器间的依赖关系、外部私有容器仓库的访问等，遇到这些高级配置，可以使用 JSON 文件来定义任务。

先用 AWS CLI 的命令来生成模板：`aws ecs register-task-definition --generate-cli-skeleton`， 之后再填入需要的信息。具体的字段的含义参见[Task Definition Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html).

例如容器间的依赖关系可以定义为：
```json
{
    ...
    "containerDefinitions": [
        {
            ...
            "dependsOn": [
                {
                    "containerName": "${name_of_a_container}",
                    "condition": "[START|STOP|SUCCESS|HEALTHY]"
                }
            ],
            ...
        }
    ],
    ...
}
```

# 部署服务

接下来需要添加服务（Service）或者任务（Task），这个过程相当于 Task definition 的实例化，即让 docker image 运行在 container 里。服务和任务的区别是前者是持续运行的，后者是一次性的。

在添加服务的过程中，需要选择运行在哪个集群上，配置部署的方案，选择是否需要让该服务被集群中其他服务发现，配置网络、负载均衡、自动增减容器实例、任务在集群上的分配方式，以及选择是否添加标签。

当服务定义好了，会自动开始在指定的集群中部署。部署完成后，就可以正常访问服务了。

# 错误排查

ECS 中提供了几种记录 log 的方式：
1. 集群级别的 Container Insights，用来监测集群的运行状况
2. 服务级别的 CloudWatch，用来监测服务的运行状况
3. 部署服务时，可以通过 CloudFormation 来监测部署的情况
4. Container 级别的 Log，可以在定义 Task definition 时，添加监测的设置。Container 运行时的 log 就可以通过 CloudWatch、Kinesis Firehose、Kinesis Data Stream、以及存入 S3 的方式进行记录
5. CloudTrails 可以记录 API 的调用情况。
