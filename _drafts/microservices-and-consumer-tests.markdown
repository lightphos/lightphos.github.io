---
layout: post
title: Consumer Tests with Pact, Junit5, SpringBoot
---

> Do you set your house on fire to test your smoke alarm?.... 
> https://docs.pact.io
> 


A Consumer creates the consumer tests which confirms the contract between the consumer and a provider it is dependant on.


Pact is consumer driven - that is, it forms part of the consumers tests.

The Pact framework creates a pact file based on interactions defined by the consumer. This is then used to verify the provider of the service interactions. The verfication is done by the Pact framework. This allows independent testing of the consumer and provider without having to integrate both together. This is a great benefit when you have numerous microservices.

The following makes use of with Java, Spring and Junit5.

**Consumer End**

https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-consumer-junit5

POM
```
        <dependency>
            <groupId>au.com.dius</groupId>
            <artifactId>pact-jvm-consumer-junit5_2.12</artifactId>
            <version>3.5.24</version>
        </dependency>
        
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <systemPropertyVariables>
                        <pact.rootDir>target/pacts</pact.rootDir>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
        </plugins>
    </build>

```

Test Example
```
import au.com.dius.pact.consumer.MockServer;
import au.com.dius.pact.consumer.Pact;
import au.com.dius.pact.consumer.dsl.PactDslWithProvider;
import au.com.dius.pact.consumer.junit5.PactConsumerTestExt;
import au.com.dius.pact.consumer.junit5.PactTestFor;
import au.com.dius.pact.model.RequestResponsePact;
import org.apache.http.HttpResponse;
import org.apache.http.client.fluent.Request;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.core.IsEqual.equalTo;


@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "test_provider", port = "1234")
public class ATest {


    @Pact(provider="test_provider", consumer="test_consumer")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        return builder
                .given("test state")
                .uponReceiving("ExampleJavaConsumerPactTest test interaction")
                .path("/")
                .method("GET")
                .willRespondWith()
                .status(200)
                .body("{\"responsetest\": true}")
                .toPact();
    }

    @Test
    void test(MockServer mockServer) throws Exception {
        HttpResponse httpResponse = Request.Get(mockServer.getUrl() + "/").execute().returnResponse();
        assertThat(httpResponse.getStatusLine().getStatusCode(), is(equalTo(200)));
    }
}

```

Pacts File
```
{
    "provider": {
        "name": "test_provider"
    },
    "consumer": {
        "name": "test_consumer"
    },
    "interactions": [
        {
            "description": "ExampleJavaConsumerPactTest test interaction",
            "request": {
                "method": "GET",
                "path": "/"
            },
            "response": {
                "status": 200,
                "body": {
                    "responsetest": true
                }
            },
            "providerStates": [
                {
                    "name": "test state"
                }
            ]
        }
    ],
    "metadata": {
        "pactSpecification": {
            "version": "3.0.0"
        },
        "pact-jvm": {
            "version": "3.5.24"
        }
    }
}
```

Run the test
`mvn clean test`

This should create the pack file above in directory target/pacts

**Producer End**

POM
```
        <dependencies>
        <dependency>
            <groupId>au.com.dius</groupId>
            <artifactId>pact-jvm-provider-junit5_2.12</artifactId>
            <version>3.5.24</version>
            <exclusions>
                <exclusion>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.8.RELEASE</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.0</version>
                <dependencies>
                    <dependency>
                        <groupId>org.junit.platform</groupId>
                        <artifactId>junit-platform-surefire-provider</artifactId>
                        <version>1.2.0</version>
                    </dependency>
                    <dependency>
                        <groupId>org.junit.jupiter</groupId>
                        <artifactId>junit-jupiter-engine</artifactId>
                        <version>5.2.0</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <systemPropertyVariables>
                        <pact.rootDir>target/pacts</pact.rootDir>
                    </systemPropertyVariables>
                </configuration>
            </plugin>


        </plugins>
    </build>


```

Copy the pact file from the consumer (for now) on to the producer projects module target directory

Test code:

```

@Provider("test-provider")
@PactFolder("target/pacts")
public class ProducerTest {

    private String providerUrl = "http://localhost:8080/service/";


    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context)
    {
        context.verifyInteraction();
    }


    @BeforeEach
    void before(PactVerificationContext context) throws Exception {
        context.setTarget(HttpTestTarget.fromUrl(new URL(
                providerUrl)));
    }

    @State({"test state"})
    public void toState() {
    }

```

then run

`mvn clean test`

**(you will need to first run the provider service)**

Output should be verified like this:

```

Verifying a pact between consumer and test_provider
  Given test state
  GET REQUEST
    returns a response which
      has status code 200 (OK)
      includes headers
        "Content-Type" with value "application/json" (OK)
      has a matching body (OK)

```

Using SpringBootTest (running a test spring runner) so you do not need to separately run the service:

```
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = Application.class, webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT,
        properties = "server.port=8080")


@Provider("test-provider")
@PactFolder("src/test/resources/pacts")
public class SpringProducerTest {

    @MockBean
    private Service service;


    @BeforeEach
    void setupTestTarget(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", 8080, "/context"));
    }

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }

    @State({"test State"})
    public void toState() {
        when(service.get(anyString()).thenReturn("dummy");
    }

}

```
