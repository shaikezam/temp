# Spring Cloud Stream Imperative Programming
*13-12-2021 - Shai Zambrovski*

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
### Spring Cloud Stream Concepts
- Bindings - an interface in which we declare our IO channels.
- Binder - the message broker platform; in our case, RabbitMQ
- Output - An interface method taht decare with `@Output`, it takes an `Object`, serializes it and then publishes it to the output channel.
- Input - Used to consume message from queue.
- Channel - Represents an input\output pipe between the our application and the message broker.

Our simple application will be a `Spring Boot Application` that publish and consume tasks.

The maven project will be defined in the `pom.xml`:
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
    <groupId>org.spring.cloud.stream</groupId>
    <artifactId>rabbitmq-imperative-programming</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>>rabbitmq-imperative-programming</name>
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
Then, let's create our model: `Task`:
```java
import org.apache.commons.lang3.RandomStringUtils;

public class Task {

    private String id;
    private String title;
    private String description;

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

    @Override
    public String toString() {
        return "Task{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", description='" + description + '\'' +
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
Now we will define our bindings interface (which responsible the Input\Output methods):
```java
import org.springframework.cloud.stream.annotation.Input;
import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.SubscribableChannel;

public interface TaskBinding {
    String TASK_CHANNEL_INPUT = "task-channel-in";
    String TASK_CHANNEL_OUT = "task-channel-out";

    @Input(TASK_CHANNEL_INPUT)
    SubscribableChannel inboundTasks();
    @Output(TASK_CHANNEL_OUT)
    MessageChannel outboundTasks();
}
```
We define the output channel named as `task-channel-out` and the input channel named as `task-channel-in`

Then, we will define our `application.properties` file:

    server.port=333
    spring.rabbitmq.host=127.0.0.1
    spring.rabbitmq.port=5672
    spring.rabbitmq.username=guest
    spring.rabbitmq.password=guest
    spring.cloud.stream.bindings.task-channel-in.destination=imperative-programming-tasks
    spring.cloud.stream.bindings.task-channel-out.destination=imperative-programming-tasks
    
    logging.level.org.springframework.cloud.stream=debug

We set our `RabbitMQ` server properties and associate between the channel and the queue to be used

Last step, we will create our `Spring Boot Application` class with the relevant annotations:
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

@SpringBootApplication
@EnableBinding(TaskBinding.class)
@EnableScheduling
public class SpringCloudStreamRabbitMQImperativeProgramming {

    private MessageChannel messageChannel;

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudStreamRabbitMQImperativeProgramming.class, args);
    }

    public SpringCloudStreamRabbitMQImperativeProgramming(TaskBinding taskBinding) {
        this.messageChannel = taskBinding.outboundTasks();
    }

    @Scheduled(fixedDelay = 1000)
    public void publishTask() {
        Task task = new Task.TaskBuilder().build();
        System.out.println("producing task: " + task);
        messageChannel.send(MessageBuilder.withPayload(task).build());
    }

    @StreamListener(TaskBinding.TASK_CHANNEL_INPUT)
    public void consumeTask(String msg) {
        System.out.println("consumed task: " + msg);
    }

}
```
Basically, our application run as a scheduled app in which each 1 seconds, it will publish to the queue a random Task instance. let's do quick explanation on the new annotations.

- `@EnableBinding` annotation must point to the binding interface.
- `@StreamListener` must point to the input channel which we define in the binding interface.

Now, if we run the application we will see in the console the prints upon tasks producing and consuming:

    producing task: Task{id=NAk32IfCnB, title='jO5Be3cXmW', description='TslmrawNbQ'}
    consumed task: {"id":"NAk32IfCnB","title":"jO5Be3cXmW","description":"TslmrawNbQ"}
    
In new versions of `Spring Cloud Stream` we noticed that some of the annotations are [deprecated](https://spring.io/blog/2019/10/17/spring-cloud-stream-functional-and-reactive "deprecated").

In the next artical we will using `Spring Cloud Stream` with `Functional Programing`

Feel free to look in the [source code](https://github.com/shaikezam/spring-cloud-stream-rabbitmq-imperative-programming "source code") and try it your own.