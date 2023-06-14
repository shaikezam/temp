# Spring Cloud Stream Functional Programming
*22-12-2021 - Shai Zambrovski*

------------
## Microservices Asynchronous Communication
There are two concepts regarding communication between microservices:

- Synchronized - via `HTTP(S)` over `REST` for example.
- Asynchronous (or, Event Driven Communication) - via message brokers for example (Kafka\RabbitMQ...)

With event driven communication, we can send from P2P or via pub\sub any events we want, for example: state changed or content.
## Spring Cloud Stream
Among the many projects within the [Spring Cloud Project](https://spring.io/projects/spring-cloud "Spring Cloud Project"), the [Spring Cloud Stream](https://spring.io/projects/spring-cloud-stream "Spring Cloud Stream") library provided the developer an abstraction\wrapper implementation on top of the messaging brokers.
That means that Spring application could:

- Conenct to the message brokers with minimal configurations.
- Replace the message broker with minimal modifications.

### RabbitMQ
We will not cover and learn about the `RabbitMQ` message broker, but it widely popular lightweight messaging platform.

We will install via docker command (if you don't want use docker, feel free to install it as you wish)

    docker run -d -p 5672:5672 -p 15672:15672 --name my-rabbit rabbitmq:3-management
    
Using this command, we installed (as `container`) a RabbitMQ `docker image` with management GUI and it will be accessible via `localhost:15672` with username `guest` and password `guest` (those are the defaults).
### Spring Cloud Stream Functional Programming vs Imperative Programming
Until version v2.1.0, SCS (Spring Cloud Stream) worked as annotation-based programing (just as the previous guide).

From this version, SCS preferred way of work is using the function based.

Both are valid and fully functioning implementations.

Both do the same thing and both produce the same result – except that, in the annotation-based, the user has to be aware of SCS abstractions (that is, messaging, channels, binding, and so on) while the actual user code has nothing to do with any of them.

Instead of working with annotation-based configuration, spring now uses detected beans of Consumer/Function/Supplier to define your streams for you. 

The `Supplier` works as publisher, the `Function` worked as Processor and the `Consumer` works as subscriber.

Our simple application will be a Spring Boot application that publish and consume tasks.

The maven project will be defined in the pom.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.6</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>shaike.zam.spring.cloud.stream</groupId>
	<artifactId>rabbitmq-hello-world-functional-programming</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>rabbitmq-hello-world-functional-programming</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>11</java.version>
		<spring-cloud.version>2020.0.4</spring-cloud.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.9</version>
		</dependency>

	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
It's pretty simillar to the Spring Cloud Stream Imperative Programming guide.

Then, let's create our model: `Task`
```java
public class Task {

    private String id;
    private String title;
    private String description;
    private Status taskStatus;

    public Task(String id, String title, String description) {
        this.id = id;
        this.title = title;
        this.description = description;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Status getTaskStatus() {
        return taskStatus;
    }

    public void setTaskStatus(Status taskStatus) {
        this.taskStatus = taskStatus;
    }

    @Override
    public String toString() {
        return "Task{" +
                "id='" + id + '\'' +
                ", title='" + title + '\'' +
                ", description='" + description + '\'' +
                ", taskStatus=" + taskStatus +
                '}';
    }

    public static class TaskBuilder {

        private String id = RandomStringUtils.randomAlphanumeric(10);
        private String title = RandomStringUtils.randomAlphanumeric(10);
        private String description = RandomStringUtils.randomAlphanumeric(10);

        public TaskBuilder() {
        }

        public TaskBuilder withId(String id) {
            this.id = id;
            return this;
        }

        public TaskBuilder withTitle(String title) {
            this.title = title;
            return this;
        }

        public TaskBuilder withDescription(String description) {
            this.description = description;
            return this;
        }

        public Task build() {
            return new Task(id, title, description);
        }

    }
}
```
Once again it's pretty simillar to the Spring Cloud Stream Imperative Programming guide except we have a task's `Status` which represent an `enum` (the status of the task).

```java
public enum Status {
    PUBLISHED,
    PROCESSED,
    SUBSCRIBED
}
```
Next we will define our `application.yml` file.
```yaml

server:
  port: 333
spring:
  cloud:
    stream:
      function:
        definition: process;subscribe;publish
      bindings:
        publish-out-0:
          destination: publisher_channel
        subscribe-in-0:
          destination: subscriber_channel
        process-in-0:
          destination: publisher_channel
        process-out-0:
          destination: subscriber_channel

  rabbitmq:
    username: guest
    password: guest
    host: 127.0.0.1
    port: 5672
```
- At the buttom we notice the `RabbitMQ` connection details.
- `spring.cloud.stream.function.definition` is a list of the function names (`Supplier`, `Function` & `Consumer`) that you will bind to Spring Cloud Stream channels.
- `spring.cloud.stream.bindings` is where you actually bind the functions to the input and output channels.
 - For example, `process-in-0` in the example above defines a binding for function `process` that is an **input** that **subscribes** to a channel with the data it receives in the first input parameter (index 0).
 
Notice how the three bound functions, `process;subscribe;publish`, match the three functions in the next `RabbitmqHelloWorldFunctionalProgrammingApplication` class:
```java
@SpringBootApplication
public class RabbitmqHelloWorldFunctionalProgrammingApplication {

    public static void main(String[] args) {
        SpringApplication.run(RabbitmqHelloWorldFunctionalProgrammingApplication.class, args);
    }

    static class Publisher {

        @Bean
        public Supplier<Task> publish() {
            return () -> {
                Task task = new Task.TaskBuilder().build();
                task.setTaskStatus(Status.PUBLISHED);
                System.out.println("Publishing task: " + task);
                return task;
            };
        }
    }

    static class Processor {

        @Bean
        public Function<Task, Task> process() {
            return task -> {
                task.setTaskStatus(Status.PROCESSED);
                System.out.println("Processing task: " + task);
                return task;
            };
        }
    }

    static class Subscriber {

        @Bean
        public Consumer<Task> subscribe() {
            return task -> {
                task.setTaskStatus(Status.SUBSCRIBED);
                System.out.println("Subscribed task: " + task);
            };
        }
    }

}
```
`publish` method binds to the `publisher_channel` to which it is going to send a random task every second. This, by the way, is a property of Spring’s implementation of the `Supplier` interface; Spring triggers it automatically every second, so it’s a great tool for testing and developing streams.

`process` receives the random task from the `publisher_channel` channel, set it's new status, and publishes the updated task on the `subscriber_channel` channel.

`subscribe` listens to the `subscriber_channel` channel and logs the task.

Now, if we run the application we will see in the console the prints upon tasks producing and consuming:

    Publishing task: Task{id='3RUwneu0f2', title='gqL2qesecR', description='2yMyOFeTWY', taskStatus=PUBLISHED}
    Processing task: Task{id='3RUwneu0f2', title='gqL2qesecR', description='2yMyOFeTWY', taskStatus=PROCESSED}
    Subscribed task: Task{id='3RUwneu0f2', title='gqL2qesecR', description='2yMyOFeTWY', taskStatus=SUBSCRIBED}
    
Feel free to look in the [source code](https://github.com/shaikezam/spring-cloud-stream-rabbitmq-functional-programming "source code") and try it your own.

