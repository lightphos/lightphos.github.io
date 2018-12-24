---
layout: post
title: Serverless Development
date: '2018-07-30 08:10:32'
categories: 'Tech'
author: Satish Ramjee
tags:
- serverless
- aws
- lambda
- fn-project
---

Imagine having the ability to just write code and run it without having to worry about packaging it, adding to repositories, deploying it, provisioning/sizing servers, adding load balancers, scaling and so on. You simply write the code and it over to some 'system' which takes care of these things. That's the serverless premise. 

This blog looks at how to develop a basic service using NodeJS and Java against a few of these, namely: AWS Lamba, OpenWhisk, Fn Project. Finally we consider serverless.com which provides an abstraction layer over these different offerings.


## A New Paradigm for Software Development

>"Serverless is the future of development"
–  Werner Vogels, CTO Amazon, [AWS London Summit 2018](https://www.youtube.com/watch?v=XXbF6cUIgfs&feature=youtu.be)

### Pay Per Use
>"Whatever drink you want, no matter how many you want at the same time, in any of the closeby bars, and only pay for how much you really drank, even it was just a sip"
–  Werner Vogels, CTO Amazon

![wv-serverless](/content/images/2018/05/wv-serverless.jpg)


### Fully Managed Execution

>"Serverless computing is an architecture where code execution is fully managed by a cloud provider, instead of the traditional method of developing applications and deploying them on servers. It means developers don't have to worry about managing, provisioning and maintaining servers when deploying code. Previously a developer would have to define how much storage and database capacity would be needed pre-deployment, slowing the whole process down."
– Scott Carey, ComputerWorldUK, July 2017


## What is Serverless

The term was made popular after Amazon introduced AWS Lamda in 2014.

Mike Roberts (martinfowler.com) has written an extensive article describing [serverless architectures](https://www.martinfowler.com/articles/serverless.html) .

Sometimes also refered to as **Function as a Service** (FaaS). The difference is FaaS stands up instances instantly, within 10's of milliseconds.

**The Serverless Compute Manifesto**
- Functions are the unit of deployment and scaling.
- No machines, VMs, or containers visible in the programming model.
- Permanent storage lives elsewhere.
- Scales per request; users cannot over- or under-provision capacity.
- Never pay for idle (no cold servers/containers or their costs).
- Implicitly fault-tolerant because functions can run anywhere.
- BYOC - Bring Your Own Code.
- Metrics and logging are a universal right.

More at [demystifying serverless compute manifesto](https://medium.com/@denismakogon/demystifying-serverless-compute-manifesto-ee07a3c2565c)


## Infrastructure as a Service or Function as a Service

We shall look briefly at the following:
- AWS Lambda
- Oracle Fn Project
- IBM Cloud Functions (Apache OpenWhisk)
- Serverless with AWS Lambda and Fn

AWS Lambda is a full FaaS. Fn has a container native approach based on docker. IBM Cloud Functions is based on OpenWhisk and provides a full FaaS.

Both Fn and OpenWhisk are open source and can be provisioned by themselves if desired on IaaS cloud providers including IBM Bluemix, Amazon EC2, Microsoft Azure, Redhat OpenShift and Google Cloud Functions. 

A FaaS however, will give you the pay per invocation benefit. Currently there is no Fn Project FaaS provider (Oracle?), however there is capability to run it so that resources are consumed only when needed. 

For a list of FaaS providers see https://github.com/faas-lane/FaaS-Lane/tree/master/candidates

<hr/>

## AWS Lambda

Let's have a go with Java and NodeJs.

### Java Lambda

On AWS, create a AWS Lambda function.

![ll1](/content/images/2018/05/ll1.png)

This will create a default hello world example.

Entry point is defined in the Handler box:
`example.Hello::myHandler`

Click the test button, add a hello world default test event with any name eg test. Create and click test with the test events selected.

You should see the following:
![ll2](/content/images/2018/05/ll2.png)


From actions -> export function -> download deployment package.

The generated class looks like so:
```
package example;

public class Hello {
    public Hello() {
    }

    public String myHandler() {
        return "Hello, World!";
    }
}
```

Let's write our own, note the lamda imports and custom User input class:
```
package com.demo.serverless.awslambda;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.LambdaLogger;

public class HiEarth {
    public String handle(User input, Context context) {
        LambdaLogger logger = context.getLogger();
        logger.log("Input : " + input);
        return "Hi earth " + input;
    }
}

public class User {
   private String input;
    public String getInput() { return input; }
    public void setInput(String input) { this.input = input; }

    @Override
    public String toString() {
        return input;
    }
}
```
The maven POM:
```

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo.serverless.awslambda</groupId>
    <artifactId>hiearth</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-lambda-java-core</artifactId>
            <version>1.1.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    
</project>
```


Run the mvn package: 
`mvn package shade:shade`
and upload the shaded jar file from the AWS Lambda console.
Change the handler info to 
`com.demo.serverless.awslambda.HiEarth::handle`
and click save
Configure the test event and change the json to 
```
{
  "input": "me"
}
```

Save and test, you should see the following:
![ll3](/content/images/2018/05/ll3.png)

This is a very simple example. You can wire in numerous trigger sources such as S3, DynamoDb and put the AWS API Gateway in front (see the serverless AWS Lambda example below which wires the API Gateway automatically).

### NodeJS Lambda

This is somewhat simpler than the Java version as you can write the code online on the AWS Console itself.

![node1](/content/images/2018/05/node1.png)

Online node editor provides 
![node2](/content/images/2018/05/node2.png)

Test in the same way outlined previously, this time you also get the execution results in the editor panel.

<hr/>

## Fn project

You can find details of this here:
https://github.com/fnproject
https://medium.com/fnproject/8-reasons-why-we-built-the-fn-project-bcfe45c5ae63

**Note:** Needs docker, as this leverages docker for each function (container native).

### Fn Client

Download from here
https://github.com/fnproject/cli/releases

### Fn Server

On Windows 10 (with git bash installed)

If you use fn start, you'll get the following:
```
fn start
docker: Error response from daemon: invalid mode: /app/data.
See 'docker run --help'.
2018/06/01 19:29:32 error: processed finished with error exit status 125
```

The work around is to manually  run up the docker image inside the docker toolbox terminal:

```
$ docker run --rm --name fnserver -it -v /var/run/docker.sock:/var/run/docker.sock -v ${pwd}/data:/app/data -p 8080:8080 fnproject/fnserver
```
If you've run this command before then just run fn start.

From git bash, point FN client to the docker instance, for example:
```
$ export FN_API_URL=http://192.168.99.100:8080
$ fn version
Client version:  0.4.100
Server version:  0.3.457
```

The Fn server acts like an API Gateway interacting with deployed functions. 

Let's start with NodeJs this time.

### NodeJS Fn Project
FN requires a docker registry to work against (you can install a docker container registry locally as well). For testing purposes we can set it to demo:

```
$ mkdir fnprojects
$ cd fnprojects

$ export FN_REGISTRY=fndemouser # normally would point to docker registry
$ fn init --runtime node hiearth
 
$ cd hiearth
$ ls
func.js  func.yaml  package.json  test.json
```

The actual code is in func.js (edited to say Hi Earth):
```
var fdk=require('@fnproject/fdk');

fdk.handle(function(input){
  var name = 'Earth';
  if (input.name) {
    name = input.name;
  }
  response = {'message': 'Hi ' + name}
  return response
})
```
To execute the function
```
$ fn run
Building image fndemouser/hiearth:0.0.1
{"message":"Hi Earth"}

$ export FN_API_URL=http://192.168.99.100:8080
$ fn deploy --app hiearth --local
$ fn apps list
hiearth
$ fn routes list hiearth
path            image                           endpoint
/hiearth        fndemouser/hiearth:0.0.4        192.168.99.100:8080/r/hiearth/hiearth
$ fn call hiearth hiearth
{"message":"Hi Earth"}
or
$ curl http://192.168.99.100:8080/r/hiearth/hiearth
{"message":"Hi Earth"}

$ curl -H "Content-Type: application/json" -d '{"name":"fred"}' http://192.168.99.100:8080/r/hiearth/hiearth
{"message":"Hi fred"}


$ docker ps -a
docker ps -a
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS                   PORTS                              NAMES
1ac6ba176bf7        fndemouser/hiearth:0.0.4   "node func.js"           32 seconds ago      Up 30 seconds (Paused)                                      01CEW56GB3NG8G00GZJ0000005
3be37f272693        fnproject/fnserver         "preentry.sh ./fnser…"   38 minutes ago      Up 38 minutes            2375/tcp, 0.0.0.0:8080->8080/tcp   functions

```

### Java Fn Project 

```
$ fn init --runtime java fn-java-demo
Creating function at: /fn-java-demo
Runtime: java
Function boilerplate generated.
func.yaml created.

$ ls
func.yaml  pom.xml  src/
```
Code is generated as shown:
```
package com.example.fn;

public class HelloFunction {

    public String handleRequest(String input) {
        String name = (input == null || input.isEmpty()) ? "world"  : input;

        return "Hello, " + name + "!";
    }

}
```
The yaml file
```
$ cat func.yaml
name: fn-java-demo
version: 0.0.1
runtime: java
cmd: com.example.fn.HelloFunction::handleRequest
build_image: fnproject/fn-java-fdk-build:jdk9-1.0.59
run_image: fnproject/fn-java-fdk:jdk9-1.0.59
format: http
```
Test it
```
$ fn run
Building image fndemouser/fn-java-demo:0.0.1 ..............................................
Hello, world!
```
Deploy to local FN server
```
$ fn deploy --local --app fn-java
Deploying fn-java-demo to app: fn-java at path: /fn-java-demo
Bumped to version 0.0.2
Building image fndemouser/fn-java-demo:0.0.2
Updating route /fn-java-demo using image fndemouser/fn-java-demo:0.0.2...

$ fn list routes fn-java
path            image                           endpoint
/fn-java-demo   fndemouser/fn-java-demo:0.0.2   192.168.99.100:8080/r/fn-java/fn-java-demo

$ echo people | fn call fn-java fn-java-demo
Hello, people

$ curl --data 'Earth' http://192.168.99.100:8080/r/fn-java/fn-java-demo
Hello, Earth!
```

BTW no need to build a jar etc. Fn does all that behind the scenes.

### Fn UI

```
$ docker run --rm -it --link fnserver:api -p 4000:4000 -e "FN_API_URL=http://api:8080" fnproject/ui

> FunctionsUI@0.0.26 start /app
> node server

WARNING: NODE_ENV value of 'production' did match any deployment config file names.
WARNING: See https://github.com/lorenwest/node-config/wiki/Strict-Mode
info: Using API url: api:8080
info: Server running on port 4000
```
On your browser head to:
```
http://localhost:4000/#/app/hiearth
```

### Other Fn features


#### Fn JRestless


>JRestless allows you to create FaaS applications using JAX-RS. We are adding support for using JRestless on Fn.

>This means you can use all the JAX-RS features you're used to, @Path, @GET, @QueryParam - all the marshalling and content-types, all the routing. All of it, in a FaaS function. JRestless uses Jersey internally so you have the full capability of the reference JAX-RS implementation.


#### Fn Flow

>Fn Flow lets you build long-running, reliable and scalable functions using Fn that only consume compute resources when they have work to do and are written purely in code. Flow supports building complex parallel processes that are readable, testable (including via unit testing) using standard programming tools. 

#### Fn LB

>The Fn Load Balancer (Fn LB) allows operators to deploy clusters of Fn servers and route traffic to them intelligently. Most importantly, it will route traffic to nodes where hot functions are running to ensure optimal performance, as well as distribute load if traffic to a specific function increases. It also gathers information about the entire cluster which you can use to know when to scale out (add more Fn servers) or in (decrease Fn servers).

#### More here
https://developer.oracle.com/opensource/serverless-with-fn-project

<hr/>

## IBM Cloud Functions

This is based on [Apache OpenWhisk](https://openwhisk.apache.org) which makes use of nginx, kafka, couchdb and docker technologies.

![openwhisk-arch](/content/images/2018/07/openwhisk-arch.png)

### Getting Started
https://console.bluemix.net/docs/openwhisk/index.html#getting-started-with-openwhisk

This offers options to code in Node, Swift, Python and PHP. For Java (and Go) we need to build locally through the CLI, see below.

### NodeJS OpenWhisk
Select from the LHS cloud menu:
functions -> start creating -> quick start templates -> hello world.

![action-helloworld](/content/images/2018/06/action-helloworld.png)

Follow the steps and it will give you an endpoint which you can call:

```
$ curl -u ...s3f1IO3Ib5J2SwowJ0tDQjgLYa1PP6jsTkmFEP -X POST
 https://openwhisk.eu-gb.bluemix.net/api/v1/namespaces/_dev/actions/hello-world/helloworld?blocking=true
 
{"duration":4,"name":"helloworld","subject":"","activationId":"9d04b2714eb94af684b2714eb94af670","publish":false,"annotations":[{"key":"limits","value":{"timeout":60000,"memory":256,"logs":10}},{"key":"path","value":"/hello-world/helloworld"},{"key":"kind","value":"nodejs:8"},{"key":"waitTime","value":38}],"version":"0.0.1","response":{"result":{"greeting":"Hello stranger!"},"success":true,"status":"success"},"end":1527855805745,"logs":[],"start":1527855805741,"namespace":"_dev"}
```

### Java OpenWhisk Bluemix CLI

For this you need to download the bluemix CLI, this is on Ubuntu 16.04.3 with WSL (Windows 10). 

```
$ curl -fsSL https://clis.ng.bluemix.net/install/linux | sh
$ bx plugin install Cloud-Functions -r Bluemix
```
Login to bluemix cloud
```
$ bx login -a api.eu-gb.bluemix.net -o "satish..." -s "dev"
API endpoint:      https://api.eu-gb.bluemix.net
Region:            eu-gb
User:              satish....
Account:            (ad7ee8767a67ae8687e7a5d3a3e3442f)
Resource group:    Default
CF API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.92.0)
Org:               satish...
Space:             dev
```
Test with the echo function
```
$ bx wsk action invoke /whisk.system/utils/echo -p message hi --result
{
    "message": "hi"
}
```

Ok now we are ready for some Java code...
```
package com.demo.serverless.bx;

import com.google.gson.JsonObject;

public class HiEarth {
    public static JsonObject main(JsonObject args) {
        String name = "stranger";
        if (args.has("name"))
            name = args.getAsJsonPrimitive("name").getAsString();
        JsonObject response = new JsonObject();
        response.addProperty("greeting", "Hi " + name + "!");
        return response;
    }
}
```
Maven POM
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo.serverless</groupId>
    <artifactId>bx-java-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.5</version>
        </dependency>
    </dependencies>

</project>
```
Package this to create a jar:
`mvn package`
Create an OpenWhisk action:
```
$ bx wsk action create hiEarthJava bx-java-demo/target/bx-java-demo-1.0-SNAPSHOT.jar --main com.demo.serverless.bx.HiEarth
ok: created action hiEarthJava
$ bx wsk action list actions
/satish.._dev/hiEarthJava                              private java

$ bx wsk action invoke hiEarthJava --result --param name Earth
{
    "greeting": "Hi Earth!"
}

$ bx wsk action get hiEarthJava --url
ok: got action hiEarthJava
https://openwhisk.eu-gb.bluemix.net/api/v1/namespaces/satish..._dev/actions/hiEarthJava
```
Remote access
```
$ curl -u ...RvI5rgPPMrs3f1IO3Ib5J2SwowJ0tDQjgLYa1PP6jsTkmFEP -X POST https://openwhisk.eu-gb.bluemix.net/api/v1/namespaces/satishramjee%40hotmail.com_dev/actions/hiEarthJava?blocking=true
{"duration":4,"name":"hiEarthJava","subject":"satish..","activationId":"88105b0fbe7245ab905b0fbe72a5ab45","publish":false,"annotations":[{"key":"limits","value":{"timeout":60000,"memory":256,"logs":10}},{"key":"path","value":"satish.._dev/hiEarthJava"},{"key":"kind","value":"java"},{"key":"waitTime","value":31}],"version":"0.0.1","response":{"result":{"greeting":"Hi stranger!"},"success":true,"status":"success"},"end":1527874719735,"logs":[],"start":1527874719731,"namespace":"satish.._dev"}
```

<hr/>

## Serverless (SLS)
This open source project (https://serverless.com), provides an abstraction layer over the various FaaS offerings, where functions can be deployed to various cloud providers: 

![serverless-providers](/content/images/2018/06/serverless-providers.png)

#### Install Serverless
```
$sudo npm install -g serverless
```


### NodeJS/AWS Lambda with SLS

Create IAM credentials:
https://serverless.com/framework/docs/providers/aws/guide/credentials/

Configure AWS credentionals:

```
serverless config credentials --provider aws --key AKI.. --secret 2z...
```

```
$ serverless create --template aws-nodejs --path sls-aws-demo
Serverless: Generating boilerplate...
Serverless: Generating boilerplate in "/mnt/c/sls-aws-demo"
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v1.27.3
 -------'

Serverless: Successfully generated boilerplate for template: "aws-nodejs"

$ cd sls-aws-demo
$ serverless deploy
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
.....
.....
Service Information
service: sls-aws-demo
stage: dev
region: us-east-1
stack: sls-aws-demo-dev
api keys:
  None
endpoints:
  None
functions:
  hello: sls-aws-demo-dev-hello
```

Generated code:
```
more handler.js
'use strict';

module.exports.hello = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Go Serverless v1.0! Your function executed successfully!',
      input: event,
    }),
  };

  callback(null, response);

  // Use this code if you don't use the http event with the LAMBDA-PROXY integration
  // callback(null, { message: 'Go Serverless v1.0! Your function executed successfully!', event });
};
```

And on AWS Lambda console:

![aws-lamda](/content/images/2018/07/aws-lamda.PNG)


Invocation:
```
serverless invoke -f hello -l
{
    "statusCode": 200,
    "body": "{\"message\":\"Go Serverless v1.0! Your function executed successfully!\",\"input\":{}}"
}
--------------------------------------------------------------------
START RequestId: 3ea2b400-9296-11e8-902a-4b7406b8b13c Version: $LATEST
END RequestId: 3ea2b400-9296-11e8-902a-4b7406b8b13c
REPORT RequestId: 3ea2b400-9296-11e8-902a-4b7406b8b13c  Duration: 1.59 ms       Billed Duration: 100 ms         Memory Size: 1024 MB    Max Memory Used: 21 MB
```


### NodeJS/Fn Project with SLS

You need Fn server running locally and you need a docker hub account or you can run a private  local registry (see https://docs.docker.com/registry/deploying/#basic-configuration and https://github.com/fnproject/fn/blob/master/docs/operating/private_registries.md).

```
$ serverless create --template fn-nodejs --path sls-fn-demo
$ cd sls-fn-demo/
```
Set up Fn variables and node modules
```
$ npm install
$ export FN_REGISTRY=<docker username> 
$ docker login 
or if you are running registry locally: 
$ export FN_REGISTRY=localhost:5000/sls
```
otherwise you'll get 
Cannot read property 'endsWith' of undefined when you deploy.

Start and point to the FN server:
```
$ fn start
on a separate terminal, depending on your docker behaviour:
$ export FN_API_URL=http://192.168.99.100:8080 
or 
$ export FN_API_URL=http://localhost:8080
```
Fire it up with serverless
```
$ serverless deploy
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: hello-world/hello deployed..
```

If you get:
```
 throw new Error(e);
          ^
Error: docker command failed
```

You need to make suer the Fn server is running (fn start):

Call it with:
```
serverless invoke --function hello --data '{"name":"People"}' -l
Serverless: Calling Function: http://localhost:8080/r/hello-world//hello
{ message: 'Hello World' }
I show up in the logs name was: World
```

*Seems like there is a bug here, as it is not passing the data values through (need to set "Content-Type: application/json" in the header).*


Fn Server will respond:
```
time="2018-06-01T23:03:21Z" level=info msg="starting call" action="server.handleFunctionCall)-fm" app=hello-world app_id=01CEYS9C29NG8G00GZJ0000001 container_id=01CEYSBJHJNG8G00GZJ0000003 id=01CEYSENHCNG8G00GZJ0000008 route=/hello
time="2018-06-01T23:04:06Z" level=info msg="hot function terminated" app_id=01CEYS9C29NG8G00GZJ0000001 cpus= error="context canceled" format=json id=01CEYSBJHJNG8G00GZJ0000003 idle_timeout=45 image="lightphos/hello:0.0.1" memory=256 route=/hello
```

### OpenWhisk with SLS

```
$ sls create –template openwhisk-nodejs -p ow-demo
$ cd ow-demo
$ npm install serverless-openwhisk
$ npm install
$ sls deploy
$ sls invoke –f hello –l
$ echo '{"name":"there"}’ | sls invoke –f hello –l
```

There is also a helpful tutorial here which provides details:

<iframe width="560" height="315" src="https://www.youtube.com/embed/GJY10W98Itc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen>
</iframe>

<hr/>

## Observations
 
We have been looking at selected serverless offerings from a development point of view. Also we've only considered Java and NodeJs/Javascript languages.

Some observations:

- Possible vendor lock in with interconnected vendor specific services 

- http://serverless.com offers cloud agnostic solutions (interconnections) though the code is specific to the vendor.  See also https://github.com/fnproject/jrestless, which allows deployment to AWS Lamba, OpenWhisk and Fn and uses JAX-RS.

- Java is a bit more involved then NodeJS
- Deployment is one command - done

- AWS Lambda has, for good reason, the most mature and complete serverless ecosystem but it's not opensource.

- OpenWhisk is a fairly complex architecture and this may make it more brittle. Getting it to run locally proved problematic although running it on bluemix worked well for the simple case.

- Fn's idea of leveraging docker with [runc](https://blog.alexellis.io/runc-in-30-seconds/) looks very appealing, but there isn't a full FaaS provider as yet. However Fn Flow works by utilising resources only when needed and Fn Flow UI visualises flows in realtime. Fn is at the time of writing still very much bleeding edge.

- Need to consider security, debuging, monitoring, logging and tracing capabilities as a start. Also need to consider ease of connecting databases, messaging, file storage etc, and connecting other functions. This article, [30 questions to ask a serverless fanboy](https://www.iheavy.com/2017/03/13/30-questions-to-ask-a-serverless-fanboy/), provides a useful list


- Fn Project, OpenWhisk, Serverless are still in beta
- Serverless provides templates to abstract many providers although some have limited support at the moment:


> Supported templates are: "aws-nodejs", "aws-nodejs-typescript", "aws-nodejs-ecma-script", "aws-python", "aws-python3", "aws-groovy-gradle", "aws-java-maven", "aws-java-gradle", "aws-kotlin-jvm-maven", "aws-kotlin-jvm-gradle", "aws-kotlin-nodejs-gradle", "aws-scala-sbt", "aws-csharp", "aws-fsharp", "aws-go", "aws-go-dep", "azure-nodejs", "fn-nodejs", "fn-go", "google-nodejs", "kubeless-python", "kubeless-nodejs", "openwhisk-java-maven", "openwhisk-nodejs", "openwhisk-php", "openwhisk-python", "openwhisk-swift", "spotinst-nodejs", "spotinst-python", "spotinst-ruby", "spotinst-java8", "webtasks-nodejs", "plugin" and "hello-world"


The relative merits of different serverless providers is looked at by various authors for example here: http://techgenix.com/serverless-computing-comparison-guide/ 
and here: 
https://headmelted.com/serverless-showdown-4a771ca561d2. 

## Source Code
The code used for this blog is available here:
https://gitlab.com/serverless-development

