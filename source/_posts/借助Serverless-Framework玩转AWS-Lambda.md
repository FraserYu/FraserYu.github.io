---
title: 借助Serverless Framework玩转AWS Lambda
date: 2020-10-12 19:13:44
tags:
    - AWS
    - lambda
categories: [AWS]
id: aws-lambda-with-serverless-framework
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201012192901.png
description: Serverless 进一步将业务单元进行细分，职责更加单一，AWS Lambda 具有成熟的 serverless 功能，借助 Serverless Framework 可以快速开发以及部署 Serverless
keywords: AWS Lambda, lambda, serverless framework
---

## 前言

微服务架构有别于传统的单体式应用方案，我们可将单体应用拆分成多个核心功能。每个功能都被称为一项服务，可以单独构建和部署，这意味着各项服务在工作时不会互相影响



这种设计理念被进一步应用，就变成了无服务（Serverless）。「无服务」看似挺荒唐的，其实服务器依旧存在，只是我们不需要关注或预置服务器。这让开发人员的精力更集中——只关注功能实现



Serverless 的典型便是 AWS Lambda

## AWS Lambda

如果你是 Java 开发人员，你应该听说过或使用过 JDK 1.8 里面的 Lambda，但是 AWS 中的 Lambda 和 JDK 中的 Lambda **没有任何关系**

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920153741.png)



这里的 AWS Lambda 就是一种计算服务，无需预置或管理服务器即可运行代码，借助 Lambda，我们几乎可以为任何类型的应用程序或后端服务运行代码，而且完全无需管理，我们要做的只是上传相应的代码，Lambda 会处理运行和扩展 HA 代码所需的一切工作



说的直白一点

> Lambda 就好比实现某一个功能的方法 （现实中，通常会让 Lambda 功能尽可能单一），我们将这个方法做成了一个服务供调用



到这里你可能会有个困惑，Lambda 既然就是一个「方法」，那谁来调用？或怎么来调用呢？



### 如何调用 Lambda 

为了回答上面这个问题，我们需要登陆到 AWS，打开 Lambda 服务，然后创建一个 Lambda Function （hello-lambda）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920155054.png)



Lambda 既然是个方法，就要选择相应的 Runtime 环境，如下图所示，总有一款适合你的（最近在用 Node.js, 这里就用这个吧）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920155335.png)

点击右下角的 Create function 按钮进入配置页面

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920155949.png)

在上图红色框线的位置就可以配置出发 Lambda 的触发器了，点击 Add trigger

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920160214.png)

从上图可以看出，AWS 内置的很多服务都可以触发 Lambda，我在工作中常用的有：

- API Gateway （一会的 demo 会用到，也是最常见的调用方式）
- ALB - Application Loac Balancer
- CloudFront
- DynamoDB
- S3
- SNS - Simple Notification Service
- SQS - Simple Queue Service



上面只是 AWS 内置的一些服务，向下滑动，你会发现，你也可以配置很多**非 AWS** 的事件源

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920160940.png)

到这里，上面的问题你应该已经有了答案了。这里暂时先无需任何 trigger，先点击右上角的 **Test** 测试一下 Lambda

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920161609.png)

一个简单的 Lambda Function 就实现了，红色框线的 response 只是告诉大家，每个请求都会有相应的 Request ID，更有 START/END 标识快速定位 Log 内容 （可以通过 CloudWatch 查看，这里暂不展开说明）



你也可能已经开始发散你的思维了，如何运用 AWS Lambda，其实在 AWS 官网有很多样例：



### 经典案例

比如为了适应多平台图片展示，一张原始图片上传到 S3 后，会通过 Lambda resize 适应不同平台大小的图片

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920162427.png)



比如使用 AWS Lambda 和  Amazon API Gateway 构建后端，以验证和处理 API 请求，当某一个用户发布一条动态，订阅用户将收到相应的通知

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920162653.png)



接下来我们就用 Lambda 实现经典的分布式订单服务案例



## 订单服务 Demo

为了增强用户使用体验，或者为了提升程序吞吐量，亦或是为了架构设计程序解耦，考虑到以上这些情况，我们通常都会借助消息中间件来完成



假设有一常见场景，用户下订单时如果选择开具发票，则需要调用发票服务，很显然调用发票服务**不是程序运行的关键路径**，这种场景，我们就可以通过消息中间件来解耦。这里有两个服务：

1. 订单服务
2. 发票服务

如果用 Lambda 来实现两个服务，整体设计思想就是这样滴：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920173028.png)



现实中，我们不可能在 AWS console 通过点击按钮来创建各个服务的，在 AWS 实际开发中， 我们通过写 CloudFormation Template （以下会简称 CFT，其实就是一种 YAML 或者 JSON 格式的定义）来创建相关 AWS 服务，如果上述这个 Demo，从图中可以看出，我们要创建的服务还是非常多的：

- Lambda * 2
- API Gateway
- SQS

如果写 AWS 原生的 CFT，要实现的内容还是挺多的



但是...... 懒惰的程序员总是能带来很多惊喜



## Serverless Framework

写 JDBC 麻烦，就有了各种持久层框架的出现，同样写 AWS 原生 CFT 麻烦，就有了 Serverless Framework （以下会简称 SF）的出现帮助我们定义相关 Serverless 组件 （顺便问一下，GraphQL 你们有在用吗？）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920195724.png)



SF 不但简化了 AWS 原生 CFT 的编写，还简化了跨云服务的定义，就好比设计模式当中的 Facade，在上面建立了一层门面，隐藏了底部不同服务的细节，降低了跨云并用云的门槛，目前支持的云服务有下面这些

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920201023.png)



这里暂时不会对 SF 展开深入的说明，在我们的 demo 中只不过是要应用 SF 来定义



### 安装 Serverless Framework

如果你有安装 Node，那只需要一条 npm 命令全局安装即可：

```shell
npm update -g serverless
```

安装过后检查一下安装版本是否成功

```shell
sls -version
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920202514.png)

### 配置 Serverless Framework

由于要使用 AWS 的 Lambda，所以要对 SF 做基本的配置，至少要让 SF 有权限创建 AWS 服务，当你创建一个 AWS 用户时，你可以获取 AK 「access_key_id」和 SK 「secret_access_key」（**不是 SKII 哦**），其实就是一种用户名和密码形式



然后通过下面一条命令添加配置就可以了：

```bash
serverless config credentials --provider aws --key 1234 --secret 5678 --profile custom-profile
```

- --provider 云服务商
- --key 你的AK
- --secret 你的SK
- --profile 如果你有多个账户时，你可以添加这个 profile 做快速区分



运行上述命令后，就会在 ~/.aws/目录创建一个名为 credentials 的文件存储上述配置，就像这样：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920202302.png)



到这里准备工作就都完成了，开始写我们的定义就好了



### 创建 Serverless 应用

通过下面一条命令创建 serverless 应用

```shell
sls create --template aws-nodejs --path ./demo --name lambda-sqs-lambda
```

- --template 指定创建的模版
- --path 指定创建的目录
- --name 指定创建的服务名称



运行上述命令后，进入 demo 目录就是下面这个结构和内容了

```shell
➜  demo tree
.
├── handler.js
└── serverless.yml

0 directories, 2 files
```

因为我们是用 Node.js 来编写 Serverless 应用，同样在 demo 目录下执行下面命令来初始化该目录，因为我们后面要用到两个 npm package

```shell
npm init -y
```

现在的结构是这样的（其实就多了一个 package.json）：

```shell
➜  demo tree
.
├── handler.js
├── package.json
└── serverless.yml

0 directories, 3 files
```



至此，准备工作都已就绪，接下来就在 serverless.yml 中写相应的定义就可以了 （门槛很低：按照相应的 key 写 YAML 即可，是不是很简单？），打开 serverless.yml 文件来看一下，瞬间懵逼？

```yml
# Welcome to Serverless!
#
# This file is the main config file for your service.
# It's very minimal at this point and uses default values.
# You can always add more config options for more control.
# We've included some commented out config examples here.
# Just uncomment any of them to get that config option.
#
# For full config options, check the docs:
#    docs.serverless.com
#
# Happy Coding!

service: lambda-sqs-lambda
# app and org for use with dashboard.serverless.com
#app: your-app-name
#org: your-org-name

# You can pin your service to only deploy with a specific Serverless version
# Check out our docs for more details
# frameworkVersion: "=X.X.X"

provider:
  name: aws
  runtime: nodejs12.x

# you can overwrite defaults here
#  stage: dev
#  region: us-east-1

# you can add statements to the Lambda function's IAM Role here
#  iamRoleStatements:
#    - Effect: "Allow"
#      Action:
#        - "s3:ListBucket"
#      Resource: { "Fn::Join" : ["", ["arn:aws:s3:::", { "Ref" : "ServerlessDeploymentBucket" } ] ]  }
#    - Effect: "Allow"
#      Action:
#        - "s3:PutObject"
#      Resource:
#        Fn::Join:
#          - ""
#          - - "arn:aws:s3:::"
#            - "Ref" : "ServerlessDeploymentBucket"
#            - "/*"

# you can define service wide environment variables here
#  environment:
#    variable1: value1

# you can add packaging information here
#package:
#  include:
#    - include-me.js
#    - include-me-dir/**
#  exclude:
#    - exclude-me.js
#    - exclude-me-dir/**

functions:
  hello:
    handler: handler.hello
#    The following are a few example events you can configure
#    NOTE: Please make sure to change your handler code to work with those events
#    Check the event documentation for details
#    events:
#      - http:
#          path: users/create
#          method: get
#      - websocket: $connect
#      - s3: ${env:BUCKET}
#      - schedule: rate(10 minutes)
#      - sns: greeter-topic
#      - stream: arn:aws:dynamodb:region:XXXXXX:table/foo/stream/1970-01-01T00:00:00.000
#      - alexaSkill: amzn1.ask.skill.xx-xx-xx-xx
#      - alexaSmartHome: amzn1.ask.skill.xx-xx-xx-xx
#      - iot:
#          sql: "SELECT * FROM 'some_topic'"
#      - cloudwatchEvent:
#          event:
#            source:
#              - "aws.ec2"
#            detail-type:
#              - "EC2 Instance State-change Notification"
#            detail:
#              state:
#                - pending
#      - cloudwatchLog: '/aws/lambda/hello'
#      - cognitoUserPool:
#          pool: MyUserPool
#          trigger: PreSignUp
#      - alb:
#          listenerArn: arn:aws:elasticloadbalancing:us-east-1:XXXXXX:listener/app/my-load-balancer/50dc6c495c0c9188/
#          priority: 1
#          conditions:
#            host: example.com
#            path: /hello

#    Define function environment variables here
#    environment:
#      variable2: value2

# you can add CloudFormation resource templates here
#resources:
#  Resources:
#    NewResource:
#      Type: AWS::S3::Bucket
#      Properties:
#        BucketName: my-new-bucket
#  Outputs:
#     NewOutput:
#       Description: "Description for the output"
#       Value: "Some output value"

```

乍一看，你可能觉得眼花缭乱，其实这是一个相对完整的 Lambda 配置全集，我们不需要这么详细的内容，不过这个文件作为我们的参考



接下来我们就定义 demo 所需要的一切 （关键注释已经写在代码中）

```yaml
service:
  name: lambda-sqs-lambda # 定义服务的名称

provider:
  name: aws # 云服务商为 aws
  runtime: nodejs12.x # 运行时 node 的版本
  region: ap-northeast-1 # 发布到 northeast region，其实就是东京 region
  stage: dev # 发布环境为 dev
  iamRoleStatements: # 创建 IAM role，允许 lambda function 向队列发送消息
    - Effect: Allow
      Action:
        - sqs:SendMessage
      Resource:
        - Fn::GetAtt: [ receiverQueue, Arn ]
      
functions: # 定义两个 lambda functions
  order:
    handler: app/order.checkout # 第一个 lambda function 程序入口是 app 目录下的 order.js 里面的 checkout 方法
    events:	# trigger 触发器是 API Gateway 的方式，当接收到 /order 的 POST 请求时触发该 lambda function
      - http:
          method: post
          path: order

  invoice:
    handler: app/invoice.generate # 第二个 lambda function 程序入口是 app 目录下的 invoice.js 里面的 generate 方法
    timeout: 30
    events: # trigger 触发器是 SQS 服务，消息队列有消息时触发该 lambda function 消费消息
      - sqs:
          arn:
            Fn::GetAtt:
              - receiverQueue
              - Arn
resources:
  Resources:
    receiverQueue: # 定义 SQS 服务，也是 Lambda 需要依赖的服务
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:custom.conf.queueName}

# package:
#   exclude:
#     - node_modules/**

custom: 
  conf: ${file(conf/config.json)} # 引入外部定义的配置变量
```

config.json 内容仅仅定义了 queue 的名称，只是为了说明配置的灵活性

```json
{
  "queueName": "receiverQueue"
}
```

因为我们要模拟订单的生成，这里用 UUID 来模拟订单号，

因为我们要调用 AWS 服务API，所以要使用 aws-sdk，

所以要安装这两个 package （这两个理由够充分吗？）

```json
{
  "name": "lambda-sqs-lambda",
  "version": "1.0.0",
  "description": "demo for lambda",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "license": "MIT",
  "dependencies": {
    "uuid": "^8.1.0"
  },
  "devDependencies": {
    "aws-sdk": "^2.6.15"
  }
}
```

接下来，我们就可以编写两个 Lambda function 的代码逻辑了



### Order Lambda Function

订单服务很简单，接收一个下单请求，下单成功后快速返回给用户，同时将订单下单成功的消息发送到 SQS 中，供下游发票服务开具发票使用

```javascript
'use strict';

const config = require('../conf/config.json')
const AWS = require('aws-sdk');
const sqs = new AWS.SQS();
const { v4: uuidv4 } = require('uuid');

module.exports.checkout = async (event, context, callback) => {
    console.log(event)
    let statusCode = 200
    let message

    if (!event.body) {
        return {
        statusCode: 400,
        body: JSON.stringify({
            message: 'No order body was found',
        }),
        };
    }

    const region = context.invokedFunctionArn.split(':')[3]
    const accountId = context.invokedFunctionArn.split(':')[4]
    const queueName = config['queueName']

    // 组装 SQS 服务的 URL
    const queueUrl = `https://sqs.${region}.amazonaws.com/${accountId}/${queueName}`
    const orderId = uuidv4()

    try {
      	// 调用 SQS 服务
        await sqs.sendMessage({
            QueueUrl: queueUrl,
            MessageBody: event.body,
            MessageAttributes: {
                orderId: {
                    StringValue: orderId,
                    DataType: 'String',
                },
            },
        }).promise();

        message = 'Order message is placed in the Queue!';

  } catch (error) {
    console.log(error);
    message = error;
    statusCode = 500;
  }

  // 快速返回订单 ID
  return {
    statusCode,
    body: JSON.stringify({
      message, orderId,
    }),
  };
};
```



### Invoice Lambda Function

发票服务逻辑同样很简单，消费 SQS 指定队列中的消息，并将开具出的发票发送到客户订单信息的 email 中

```javascript
module.exports.generate = (event, context, callback) => {
    console.log(event)
    try {
        for (const record of event.Records) {
          const messageAttributes = record.messageAttributes;
          console.log('OrderId is  -->  ', messageAttributes.orderId.stringValue);
          console.log('Message Body -->  ', record.body);
          const reqBody = JSON.parse(record.body)
          // 睡眠 20 秒，模拟生成发票的耗时过程
          setTimeout( () => {
              console.log("Receipt is generated and sent to :" + reqBody.email)
          }, 20000)
        }
    } catch (error) {
        console.log(error);
    }
}
```

到此 demo 的代码就全部实现了，从中你可以看到：

> 我们没有关注 lambda 的底层服务细节，没有关注 sqs 的服务，只是简单的代码逻辑实现以及服务之间的串联定义

最后我们看一下整体的目录结构吧：

```shell
.
├── app
│   ├── invoice.js
│   └── order.js
├── conf
│   └── config.json
├── package.json
└── serverless.yml

2 directories, 5 files
```

## 发布 Lambda 应用

在发布之前，编译一下应用，安装必须的 package「uuid 和 aws-sdk」

```shell
npm install
```

发布应用非常简单，只需要一条命令：

```shell
sls deploy -v
```

运行上述命令后大概需要等带几十秒钟, 在构建的最后，会打印出我们的构建服务信息：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920220218.png)

上图的 endpoints 就是我们一会要访问的 API gateway 触发 lambda 的入口，在调用之前，我们先到 AWS console 看一下我们定义的服务



### lambda functions

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920220437.png)



### SQS-receverQueue

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920220559.png)



### API Gateway

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920221027.png)



### S3

从上图的构建信息中你应该还看到一个 S3  bucket 的名称，我们并没有创建 S3， 这是 SF 自动帮我们创建，用来存储 lambda zip package 的

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920221703.png)



### 测试

调用 API gateway 的 endpoint 来测试 lambda

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920223806.png)



打开 SQS 服务，你会发现，接收到一条消息：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920231011.png)

接下来我们看看 Invoice Lambda function 的消费情况，打开 CloudWatch 查看 log：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200920232427.png)

从 log 中可以看出程序“耗费” 20 秒后打印了向客户邮件的 log（邮件也可以借助 AWS SES 邮件服务来实现）



至此，一个完整的 demo 就完成了，实际编写的代码并没有多少，就搞定了这么紧密的串联



### 删除服务

Lambda 是按照调用次数进行收取费用的，为了防止造成额外的开销，demo 结束后通常都会将服务销毁，使用 SF 销毁刚刚创建的服务也非常简单，只需要在 serverless.yml 文件目录执行这条命令：

```shell
sls remove
```



## 总结与感受

AWS Lambda 是 Serverless 的典型，借助 Lambda 可以实现更小粒度的“服务”，无需服务搭建也加快了开发速度。Lambda 同样可以结合 AWS 很多其服务，接收请求，将计算结果传递给下游服务等。另外很多第三方合作伙伴也在加入 Lambda 的 trigger 大部队，给 Lambda 更多触发可能，同时，借助 CI/CD，可以快速实现功能闭环



开通 AWS free tier，足够你玩转 Lambda, 公众号回复【lambda】获取代码地址

