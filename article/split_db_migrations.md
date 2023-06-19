# Split DB migration from services using Kubernetes jobs
*15-02-2022 - Shai Zambrovski*

------------
## Motivation
While in the monolithic architecture we perform a DB migration \ upgrade before the application is started, in the micro-services pattern this can lead to some huge issues:
- Upon each new service is created or restarted, it will try to run the migrations again.It's fine if our migration scripts are written correctly and this should not be an issue however it’s not a best practice & clean design.
- If the migration takes a while, the service can act as not-healty and to be terminated before the migration is done.it can be resolved while increasing the initial delay for the service's readiness check, but then it makes it hard to understand if a service is not starting because it’s of DB migrations or because of a different issue.
- Depending on our deployment strategy (how many services can be added/removed at any point of time during a deployment), we may have 2 identical migrations running at the same time, resulting in conflicts and errors.

To handle those issues we must separate the migration part from the services itself.

## The Solution - K8S Jobs
To demonstrate our solution, we will using Kubernetes as a services orchestration (if you are not familiar with K8S, please read about it, it must have in your tool box).
![](https://shaikezam.com/style/split_db_migration.png)

In our scenario, we have a simple application, that runs inside K8S deployment in three pods.

We attach to the app a LoadBalancer K8S service so, we can access to this deployment from outside via localhost.

Also, we have another K8S deployment that contains a single instance of Postgres pod and we attached to it a K8S ClusterIP service.

Last, we will have a K8S Job that runs an image of Flyway that responsible to perform the DB migration.

We will migrate DB using [Flyway](https://flywaydb.org/ "Flyway") - Robust schema evolution across all your environments, with ease, pleasure, and plain SQL.

In the simple Spring Boot app we will have two application.yml files for each profile:

**dev** - that we will enable Flyway migration, relevant only in local devlopement environment:

```yaml
server:
  port: 5555
spring:
  application:
    name: migration-poc

  jpa:
    database-platform: postgres
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres?currentSchema=testdb
    username: postgres
    password: blabla
  flyway:
    url: jdbc:postgresql://localhost:5432/postgres?currentSchema=testdb
    user: postgres
    password: blabla
```

**prod** - that will disable Flyway relevant only in the production environment.
```yaml
server:
  port: 5555
spring:
  application:
    name: migration-poc

  jpa:
    database-platform: postgres
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  datasource:
    url: jdbc:postgresql://postgres:5432/postgres?currentSchema=testdb
    username: postgres
    password: blabla
  flyway:
    enabled: false
```
As we can see, our app runs over port 5555 and attached to Postgres DB

Also, we will create two Dockerfile (Docker can build images automatically by reading the instructions from a Dockerfile) to create our images:

Dockerfile for building our app's image:

    FROM alpine:3.15
    RUN apk add --no-cache openjdk11
    RUN apk --no-cache add curl
    COPY target/app.jar /app/app.jar
    WORKDIR /app
    ENTRYPOINT ["java","-Dspring.profiles.active=prod","-jar","app.jar"]
    
Our base image is Alpine and we install on top of it a JDK version 11.

After that, we copy the app's Jar file and the entrypoint (an instruction is used to set executables that will always run when the container is initiated.) will start the Jar with the prod profile (to disable the Flyway)

The 2nd Dockerfile for building an image on top of Flyway:

    
    FROM flyway/flyway:8-alpine
    RUN ["rm", "-fr", "/flyway/conf"]
    RUN ["rm", "-fr", "/flyway/sql"]
    COPY flyway.conf /flyway/conf/
    COPY src/main/resources/db/migration/*.sql /flyway/sql/
    ENTRYPOINT ["flyway", "migrate"]
	
We will remove the default Flyway configuration and all upgrade scripts and copy our configuration from the root folder and the upgrade scripts from src/main/resources/db/migration/.

And finally, the entrypoint will be flyway migrate to start the migration.

## Running the POC
Clone the [github repository](https://github.com/shaikezam/k8s-split-DB-migration-from-services "github repository") and follow the readme.