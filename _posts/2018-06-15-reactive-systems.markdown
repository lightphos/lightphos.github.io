---
layout: post
title: Reactive Software Development
date: '2018-06-15 16:29:31'
categories: 'Tech'
author: Satish Ramjee
tags:
- reactive
- jetty
- netty
- vertx
- nodejs
---

## The Mobile Revolution

According to the following projection ([statista](https://www.statista.com/statistics/274774/forecast-of-mobile-phone-users-worldwide/)), the estimated number of mobile users in 2019 will be around 5 billion. 

Software applications therefore have to be designed to handle requests from a large number of devices. 

The amount of data being requested is also large due to video, audio, apps and the like. In 2018, in average users will watch 20 hours of video per month, listen to 10 hours of audio, make 11 video calls and download 20 apps.

In addition, users now expect much faster response times and 100% uptime.

The limit to meeting this demand however is not hardware but how we construct  software systems: the OS and required applications.

This limit is related to the [C10K problem](http://www.kegel.com/c10k.html). (The increasing rise in demand means that we are now having to consider the [C10M problem](http://highscalability.com/blog/2013/5/13/the-secret-to-10-million-concurrent-connections-the-kernel-i.html)).

## The C10K Problem

The C10K problem, refers to the inability of a server to scale beyond 10,000 concurrent connections/clients due to resource exhaustion. 

This is not just about response speed or requests per second (performance), but also about having the ability to handle a large number of multiple separate requests in a timely manner (scalability).

We solve this, partially, by providing web servers that are capable of handling many clients, through server socket optimisation and event driven techniques. 

## Threads and Sockets

Servers that employ the thread-per-client model, can be confounded when pooled threads spend too much time waiting on blocking operations, usually I/O. 

To assign the incoming packet to the correct thread, the OS employs a nonlinear thread search algorithm (O[n^2]) rather than constant time look up (epoll(), IOCompletionPort). This gets markedly slower with each increase in the number of threads.

Threads use up both memory (~1MB of memory per thread for stack) and CPU time (context switching).

As a result, blocking operations hamper scalability both by exhausting a server's memory and its CPU.

Resources (http/tcp connections) mean network sockets and threads. Network sockets on unix are file descriptors. Check it out with `cat /proc/sys/fs/file-max`, and ulimit -S -n (macos).

## Scalability and Performance

Performance and scalability are different concerns. The issue is scale rather than speed. 

Increasing memory or cpu doesn’t solve the scalability issue. Although the performance initially increases, at a critical number of connections, it degrades sharply. The following graphs are from Robert Grahams blog [Scalability: it's the question that drives us](https://blog.erratasec.com/2013/02/scalability-its-question-that-drives-us.html#.Wv07-Xfw-F4).

<div style="display:flex">
     <div style="flex:1;padding-right:5px;">
         
![connection1](/content/images/2018/05/connection1.png)
<center><b>Thread based</b></center>
     </div>
     <div style="flex:1;padding-left:5px;">
    
![connection2](/content/images/2018/05/connection2.png)
<center><b>Event based</b></center>
    </div>
</div>


<p/>

Doubling the server speed shows that performance has increased but at the limit of concurrent requests it degrades and does not scale.

On an event based server, on the other hand though the performance is 60% it stays constant and does not degrade as the number of connections increases. (The orange line is an event based server such as nodejs/nginx)

### Scalable System Design

C10K solutions optimised the kernel (green threads - non-kernel application threads, file descriptors, ulimits, java nio, [tcp user mode stacking](http://blog.erratasec.com/2013/02/custom-stack-it-goes-to-11.html#.WGJcc-XX9MY) etc).

The solutions make use of asynchronous event notification and constant time socket lookup (epoll).

Vertical scaling, that is, adding more CPU, RAM, HD is limited and expensive. Horizontal scaling is cheaper as commodity hardware can suffice. Although you need to be aware of the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem) limitations.

Reactive software, in essence makes use of non-blocking asynchronous request handling and is designed to be scaleable (elastic), resilient and responsive (low latency).


## Reactive Systems and Reactor

The [Reactive Manifesto](http://www.reactivemanifesto.org) presents principles for coherent reactive systems that also address the scaleability problems discussed.

Reactive Systems are responsive, resilient, elastic and message driven. The elastic aspect allowing for expansion. 

Message driven means passing asynchronous messages between components and reacting to events, hence inherently building loosely coupled and non-blocking systems.

![Reactive-Software-2](/content/images/2018/05/Reactive-Software-2.png)

Many use the [Reactor pattern](https://en.wikipedia.org/wiki/Reactor_pattern) for reactive systems and is based on event polling, a dispatcher and message handlers. 

These process requests through a single thread in an event loop, and absolve the coder from concerns about synchronisation. You do need to ensure that the event loop itself is not blocked. Any blocking calls should be spawned off in a new controlled thread, or replaced with non-blocking versions.


## Reactive with Spring Webflux, Vertx, NodeJS, Java 9

Some candidate technologies:

####Spring Webflux 
[WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html), Spring Framework 5. Can be served by Netty, Undertow etc.
Includes the concept of backpressure, through [reactive streams](https://github.com/reactive-streams/reactive-streams-jvm#reactive-streams), which is a mechanism to make sure producers do not flood consumers. 

###Vertx
Vertx is a java library that uses an event loop built on the netty library. Vertx includes an in-memory data grid tailored to allow messaging, the event bus. An interesting feature allows making use of multi-cores chips easily by setting `-Dvertx.options.eventLoopPoolSize={number of cores}`.

####NodeJS/RxJS
Written in C++, uses libenv for the event loop algorithm. Based on Google's V8 JS engine which also written in C++. Multicore utilisation is possible through a cluster module but you have to write code to make use of it.

####Java 9 Flow API
Similar to reactor pattern, Flow has a Publisher/Subscriber model.
https://community.oracle.com/docs/DOC-1006738
http://www.reactive-streams.org


## Comparison of Event Based and Thread Based Servers

Here the following are compared: Spring Webflux with Netty, Spring Jetty (traditional thread based), Vertx and NodeJs.
 
### Setup
Load test by using Apache Bench

`ab -c250 -n250 http://localhost:8080/services/`

The java based application resource usages were measured using [VisualVM](https://visualvm.github.io/download.html)

The examples were run on MacOS High Sierra with a 2.4 GHz Intel Core 2 Duo and 8GB memory. The comparison is useful in as much it highlights the relative resource usage. The file descriptor limit needs increasing for the test eg
`ulimit -S -n 1200` on the terminal running **ab**.


### Spring WebFlux Framework 5

![springimage](/content/images/2018/05/springimage.png)

This is a very simple REST service with a main thread delay to simulate a blocking call, obviously in a normal design we would avoid this and use an asynchrous version with a callback handler. Creating new threads for blocking calls would end up having the same problems describe above.

#### Maven Pom Dependencies
```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
#### Webflux code


<pre>@RestController
public class ReactiveController {

    @GetMapping("/services")
    public ResponseEntity serverResponseMono() {
        // pretend we are a long running backend call
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return ResponseEntity
                .ok()
                .header("Connection", "close")
                .contentType(MediaType.APPLICATION_JSON)
                .body(Flux.just("test"));
    }

}
    </pre>

  

#### WebFlux/Netty Performance

<div style="display:flex">
     <div style="flex:1;padding-right:5px; padding-top:22px;">

![webflux-h-x4a](/content/images/2018/05/webflux-h-x4a.png)
     </div>
     <div style="flex:1;padding-left:5px;">

![webflux-t-x4-1](/content/images/2018/05/webflux-t-x4-1.png)
    </div>
</div>

**Note:** There appears to be a bug in Apache Bench with the way it handles connections when calling a netty server. The work round was to create load by spawning multiple curls, not ideal though.

There is a small rise in memory, no rise in threads. The requests are queued in the OS due to event loop processing, hence the need for appropriately sized file descriptors (ulimit -S -n).

### Spring Jetty

A similar very simple REST code with the thread based jetty server.

#### Maven Pom Dependencies
```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
    </dependencies>
```

#### Rest API Code
```
@RestController
public class Controller {

    @RequestMapping(value="/services", method = GET)
    public String list() throws Exception {
        Thread.sleep(5000); // simulate service latency to give time for load test
        return "test";
    }
}

```
#### Spring Jetty Performance

<br/>
<div style="display:flex">
     <div style="flex:1;padding-right:5px;padding-top:22px;">
         
![jetty-heap](/content/images/2018/06/jetty-heap.png)
     </div>
     <div style="flex:1;padding-left:5px;">
    
![jetty-thread](/content/images/2018/06/jetty-thread.png)
    </div>
</div>

Here threads are created as needed
Jetty was adjusted to accept 500 connections (up from the default of 200)

### Eclipse Vertx
![vertx](/content/images/2018/05/vertx.png)

#### Maven Pom Dependencies
```
    <dependencies>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-core</artifactId>
            <version>3.3.3</version>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-web</artifactId>
            <version>3.3.3</version>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-unit</artifactId>
            <version>3.0.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
#### Vertx Code

<pre>
public class ReactVerticle extends AbstractVerticle {

    @Override
    public void start(Future<Void> fut) {

        Router router = Router.router(vertx);
        router.route("/services").handler(r -> {
            try {
                Thread.sleep(5000); // vertx will process a request at a time
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            r.response()
                .putHeader("content-type", "text/html")
                .end("Vert.x 3");
        });

        vertx
            .createHttpServer()
            .requestHandler(router::accept)
            .listen(8080, result -> {
                if (result.succeeded()) {
                    fut.complete();
                } else {
                    fut.fail(result.cause());
                }
            });
    }

}
</pre>

#### Vertx Performance

Start the app like so:
`java -jar target/vertx-demo-1.0-SNAPSHOT-fat.jar`

<div style="display:flex">
     <div style="flex:1;padding-right:5px;padding-top:22px;">
         
![vertx-heap-3x](/content/images/2018/05/vertx-heap-3x.png)
     </div>
     <div style="flex:1;padding-left:5px;">
    
 ![vertx-threads](/content/images/2018/05/vertx-threads.png)
     </div>
</div>

Vertx checks for blocking calls:

`WARNING: Thread Thread[vert.x-eventloop-thread-0,5,main] has been blocked for 3282 ms, time limit is 2000`

Again the event loop architecture processes one request at a time which is why you either need to spawn off a new thread or use non-blocking calls.

For a 2 core machine you may wish to try:
`java -jar -Dvertx.options.eventLoopPoolSize=2 target/vertx-demo-1.0-SNAPSHOT-fat.jar`



### NodeJs

Uses [memeye](https://github.com/JerryC8080/Memeye) to measure memory usage.

#### NodeJs Code
```
const http = require('http')
const port = 3000;
const memeye = require('memeye');
memeye();

const requestHandler = (request, response) => {
  setTimeout(function() {
        response.end('done ');
  }, 5000);
}

const server = http.createServer(requestHandler)
server.listen(port, (err) => {
  if (err) {
    return console.log('something bad happened', err)
  }
  console.log(`server is listening on ${port}`)
})
```

#### NodeJs Memory Usage
![nodejs](/content/images/2018/05/nodejs.png)

**RSS:** Resident Set Size, total RAM allocated to the node process.

See also [scaling nodejs](https://medium.freecodecamp.org/scaling-node-js-applications-8492bd8afadc)

## Resource Utilisation Summary

<center>
<table border=1 cellpadding=2px>
 <tr style="font-weight: bold;"><td>Technology</td><td>Approx Heap Size</td><td>Threads</td
 </tr>
 <tr><td>Netty (Java 8)</td><td> 40 Mb</td><td>17</td>
 </tr>
 <tr><td>Jetty (Java 8)</td><td> 150 Mb</td><td>274</td>
 </tr>
 <tr><td>Vertx (Java 8)</td><td> 14 Mb</td><td>19</td>
 </tr>
 <tr><td>NodeJs</td><td> 3 Mb</td><td>n/a</td>
 </tr>
</table>
</center>
  
<br/>

Observe the comparatively higher memory requirement of the Jetty based service. The thread capacity needs careful handling to manage requests.

## Source Code

The source code for this comparison can be found at https://gitlab.com/reactive-comparison


# The Changing Landscape for Scalability Solutions
## Serverless

After all that perhaps we should be investing our time in serverless applications instead, given the following statements.

```
"Serverless is the future of development"
– Werner Vogels CTO Amazon, AWS London Summit 2018

“Thanks to serverless computing we can go even further. The new world 
needs to be asynchronous and elastic by design, and OpenWhisk provides 
the features that we need.”
– Luis Enriquez, Head of Platform Engineering & Architecture, Santander Group

"In a serverless world you don’t have to think about containers any more,
you just write your code. You glue the different pieces together that are 
all managed. They are managed for you" 
– Werner Vogels, CTO Amazon
```

## Going serverless?

AWS Lamba https://aws.amazon.com/lambda/
Oracle http://fnproject.io
Apache OpenWhisk https://openwhisk.apache.org
Microsoft Azure Functions https://azure.microsoft.com/en-us/services/functions/
Google Cloud Functions https://cloud.google.com/functions/

See the blog on [Serverless Development](http://blog.ramjee.uk/2018/07/30/serverless-development/)

## References

1. http://www.kegel.com/c10k.html
2. UNIX Network Programming: Networking APIs: Sockets and XTI; Volume 1 Hardcover, January 15, 1998, W. Richard Stevens
3. https://www.forbes.com/sites/connieguglielmo/2014/02/05/mobile-traffic-will-continue-to-rise-rise-rise-as-smart-devices-take-over-the-world/#61cd6f0628a5
4. http://highscalability.com/blog/2013/5/13/the-secret-to-10-million-concurrent-connections-the-kernel-i.html
5. https://mrotaru.wordpress.com/2015/05/20/how-migratorydata-solved-the-c10m-problem-10-million-concurrent-connections-on-a-single-commodity-server/
6. http://blog.erratasec.com/2013/02/scalability-its-question-that-drives-us.html#.WGJZT-XX9MY 
7. http://blog.erratasec.com/2013/02/custom-stack-it-goes-to-11.html#.WGJcc-XX9MY
8. http://www.reactivemanifesto.org
9. https://en.wikipedia.org/wiki/Reactor_pattern
10. http://escoffier.me/vertx-hol/
11. https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/
12. http://software.schmorp.de/pkg/libev.html
13. https://medium.com/@ggonchar/reactive-spring-5-and-application-design-impact-159f79678739



