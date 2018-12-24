---
layout: post
title: Spring Cloud Integration + AWS Kinesis  + Localstack + Serverless (SAM)
---

Install pip

Install aws cli
https://aws.amazon.com/cli/
`pip install awscli`

Install localstack
https://github.com/localstack/localstack

`pip install localstack`
run it up
`localstack start`
```
Starting local dev environment. CTRL-C to quit.
Starting mock API Gateway (http port 4567)...
Starting mock DynamoDB (http port 4569)...
Starting mock SES (http port 4579)...
Starting mock Kinesis (http port 4568)...
Starting mock Redshift (http port 4577)...
Starting mock S3 (http port 4572)...
Starting mock CloudWatch (http port 4582)...
Starting mock CloudFormation (http port 4581)...
Starting mock SSM (http port 4583)...
Starting mock SQS (http port 4576)...
Starting local Elasticsearch (http port 4571)...
Starting mock SNS (http port 4575)...
Starting mock DynamoDB Streams service (http port 4570)...
Starting mock Firehose service (http port 4573)...
Starting mock Route53 (http port 4580)...
Starting mock ES service (http port 4578)...
Starting mock Lambda service (http port 4574)...
2018-08-10T17:04:52:WARNING:infra.pyc: Service "elasticsearch" not yet available, retrying...
Ready.
```

Optionally start up Localstack UI
`localstack web`

Then browse to:

`localhost:8080` 

Install awslocal
https://github.com/localstack/awscli-local

### Create Stream

Either aws point to localhost:4568 or awslocal
`aws --endpoint-url=http://localhost:4568 kinesis list-streams`

or
`awslocal kinesis list-streams`

```
{
    "StreamNames": [
    ]
}
```

awslocal kinesis create-stream --stream-name example --shard-count 1
```
{
    "StreamNames": [
        "example"
    ]
}
```
awslocal kinesis describe-stream --stream-name example




## Spring AWS Kinesis

Based on this:

https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/kinesis-samples/kinesis-produce-consume

### POM
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eg.example</groupId>
    <artifactId>example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath />
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <maven.build.timestamp.format>dd-MM-yyyy HH:mm</maven.build.timestamp.format>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <skip.surefire.tests>false</skip.surefire.tests>
        <spring-cloud-stream-binder-kinesis.version>1.0.0.M1</spring-cloud-stream-binder-kinesis.version>
    </properties>

    <repositories>
        <repository>
            <id>spring.io</id>
            <url>http://repo.spring.io/plugins-release</url>
        </repository>
    </repositories>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kinesis</artifactId>
            <version>${spring-cloud-stream-binder-kinesis.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-maven-plugin</artifactId>
                <version>1.18.0.0</version>
                <executions>
                    <execution>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>delombok</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>

    </build>
</project>
```

### application.yaml

```
originator: KinesisProducer
server:
 port: 64398
management:
 port: 8082
 context-path: /manage
security:
 enabled: false
# user:
#  name: admin
#  password: 2c76788d-e661-49fd-baba-4b41e7c1dd47


spring:
  cloud:
    stream:
      bindings:
        publicationsOutgoing:
          destination: example
          content-type: application/json
          producer: 
            partitionKeyExpression: "1" 
        publicationsIncoming:
          destination: example
          content-type: application/json

cloud:
  aws:  
    region:  
      static: us-east-1
    credentials:
      accessKey: dummy
      secretKey: dummy

# https://github.com/localstack/localstack
aws:
  endpoint:
    s3: http://localhost:4572
    kinesis: http://localhost:4568
    dynamodb: http://localhost:4569
  cbor.disable: ${AWS_CBOR_DISABLE:true}

logging: 
  level:
    com: 
      amazonaws: INFO
    org:
      apache: 
        http: INFO
```

### Config
```
@Configuration
public class AwsConfig {

    @Value("${aws.endpoint.kinesis}")
    private String kinesisEndpoint;

    @Value("${aws.endpoint.dynamodb}")
    private String dynamoDbEndpoint;

    @Value("${aws.cbor.disable}")
    private String disableCbor;

    @Bean
    public AmazonKinesisAsync amazonKinesis(AWSCredentialsProvider awsCredentialsProvider, RegionProvider regionProvider) {

        if (disableCbor != null)
            System.setProperty(SDKGlobalConfiguration.AWS_CBOR_DISABLE_SYSTEM_PROPERTY, "true"); // if using localstack

        String region = regionProvider.getRegion().getName();
        return (AmazonKinesisAsync)((AmazonKinesisAsyncClientBuilder)((AmazonKinesisAsyncClientBuilder)AmazonKinesisAsyncClientBuilder.standard()
                .withCredentials(awsCredentialsProvider))
                .withEndpointConfiguration( new AwsClientBuilder.EndpointConfiguration(kinesisEndpoint, region)))
                .build();
    }


    @Bean
    public MetadataStore kinesisCheckpointStore(AWSCredentialsProvider awsCredentialsProvider, RegionProvider regionProvider) {
        String region = regionProvider.getRegion().getName();
        AmazonDynamoDBAsync dynamoDB = (AmazonDynamoDBAsync)((AmazonDynamoDBAsyncClientBuilder)(
                (AmazonDynamoDBAsyncClientBuilder)AmazonDynamoDBAsyncClientBuilder.standard()
                        .withCredentials(awsCredentialsProvider))
                .withEndpointConfiguration( new AwsClientBuilder.EndpointConfiguration(dynamoDbEndpoint, region)))
                .build();
//        KinesisBinderConfigurationProperties.Checkpoint checkpoint = this.configurationProperties.getCheckpoint();
        DynamoDbMetaDataStore kinesisCheckpointStore = new DynamoDbMetaDataStore(dynamoDB, "checkpoint");
        kinesisCheckpointStore.setReadCapacity(1);
        kinesisCheckpointStore.setWriteCapacity(1);
        kinesisCheckpointStore.setCreateTableDelay(1);
        kinesisCheckpointStore.setCreateTableRetries(25);
        return kinesisCheckpointStore;
    }

}
```

The remain spring boot code is as in:
https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/kinesis-samples/kinesis-produce-consume

### To test

Need this for localstack (already defined in application.yaml, so can ignore if you have it in there)
```
export AWS_CBOR_DISABLE=1
```
Start the producer
```
java -jar target/example-1.0-SNAPSHOT.jar --originator=KinesisProducer --server.port=64398
```
Start the consumer
```
java -jar target/example-1.0-SNAPSHOT.jar --originator=KinesisConsumer --server.port=64399
```

Send a message:
`curl -X POST http://localhost:64399/ -H 'cache-control: no-cache' -H 'content-type: application/json' -d '{"name":"a message"}'`

If you have turned on security:

`
 curl -X POST http://localhost:64398/ -H 'authorization: Basic 2c76788d-e661-49fd-baba-4b41e7c1dd47' -H 'cache-control: no-cache' -H 'content-type: application/json' -d '{"name":"pen"}'
`

The consumer should display similar to:
```
 An event has been received Event [id=null, subject=Publication [id=25e1252e-fc2a-419f-a208-80688823ff1d, name=a message], type=EXAMPLE, originator=KinesisProducer]
```


# SAM
From here:
https://medium.com/@mengjiannlee/local-deployment-of-aws-lambda-spring-cloud-function-using-sam-local-and-localstack-dc7669110906

### Install SAM
Had to do this:
`npm config set unsafe-perm=true`
Then
`sudo npm install -g aws-sam-local`

### SQS

`awslocal sqs list-queues`
```
{
    "QueueUrls": [
        "http://localhost:4576/queue/test_queue"
    ]
}

`awslocal sqs create-queue --queue-name test_queue`

```
Send message:
` awslocal sqs send-message --queue-url http://localhost:4576/queue/test_queue --message-body "test"`
```
{
    "MD5OfMessageBody": "098f6bcd4621d373cade4e832627b4f6",
    "MD5OfMessageAttributes": "d41d8cd98f00b204e9800998ecf8427e",
    "MessageId": "d1849878-0b63-4f32-8c65-b874a2c785bb"
}
```

`awslocal sqs receive-message --queue-url http://localstack:4576/queue/test_queue`

```
{
    "Messages": [
        {
            "Body": "test",
            "ReceiptHandle": "d1849878-0b63-4f32-8c65-b874a2c785bb#fb995e59-01a0-4ee7-aa7c-616cf24e3597",
            "MD5OfBody": "098f6bcd4621d373cade4e832627b4f6",
            "MessageId": "d1849878-0b63-4f32-8c65-b874a2c785bb"
        }
    ]
}
```
Based on this code:
https://github.com/mengjiann/aws-lambda-sqs


## Serverless Offline LocalStack

https://www.npmjs.com/package/serverless-offline-localstack

npm i serverless-offline-localstack
