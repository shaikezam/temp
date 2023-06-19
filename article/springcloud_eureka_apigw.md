# Exploring Microservices Architecture with with Spring Cloud: Eureka and API Gateway
*02-06-2023 - Shai Zambrovski*

------------
In today's rapidly evolving world of software development, building scalable and resilient applications has become a necessity.

Microservices architecture offers a solution by decomposing applications into smaller, independent services that can be developed, deployed, and scaled individually.

Two key components in the microservices landscape are `Service Registry\Discovery` and `API Gateway`.
## Spring Cloud
Spring Cloud is a framework within the Spring ecosystem that provides a set of tools and libraries to simplify the development of distributed systems and microservices-based applications.

It aims to address common challenges in distributed computing, such as service discovery, configuration management, load balancing, and more.

Spring Cloud leverages popular technologies like Netflix OSS components (e.g., Eureka, Ribbon, Hystrix) and integrates them with Spring Boot to enable developers to easily build scalable, resilient, and cloud-native applications.

It promotes the use of microservices architecture and provides a consistent and opinionated approach to building distributed systems using Spring's programming model.

## Api Gateway
An API gateway is a server or service that acts as an entry point for all client requests to a microservices architecture.

It provides a centralized point of control and management for APIs, allowing for features like authentication, rate limiting, request/response transformation, and routing to multiple services.

The API gateway acts as a single interface for clients, abstracting away the complexities of the underlying microservices and providing a unified and consistent API for clients to interact with.

![](https://shaikezam.com/style/api-gateway-2.png)

### Spring Cloud API Gateway
Spring Cloud Gateway is a lightweight and highly customizable API gateway built on top of Spring Boot.

It provides a way to route and filter HTTP requests to backend services based on various criteria.

The gateway acts as a single entry point for all client requests, enabling centralized control and management of API traffic.

It supports dynamic routing, load balancing, and other advanced features to ensure reliability, scalability, and security of the microservices architecture.

Spring Cloud Gateway integrates seamlessly with other components of the Spring Cloud ecosystem, such as service discovery and configuration management, making it an ideal choice for building robust and resilient API gateways in distributed systems.

## Service Registry\Discovery
In a distributed system composed of numerous services, the ability to locate and communicate with these services is vital. Service registry and discovery play a crucial role in managing the dynamic nature of these distributed architectures.

A service registry is a central repository where services can register their location and metadata, allowing other services to discover and interact with them. It serves as a directory or catalog of available services within the system.

Service discovery, on the other hand, is the mechanism through which services can locate and connect to other services dynamically. By leveraging the service registry, services can query and retrieve the necessary information to establish communication with their dependencies.

Service registry and discovery provide scalability, fault tolerance, and flexibility in distributed systems by enabling seamless service-to-service communication without hard-coded dependencies. They facilitate dynamic updates and scaling of services, making it easier to add, remove, or replace services in the system.

### Client vs server side service discovery
Client service registry refers to the capability of a service or application to dynamically register itself with a service registry and provide its availability and metadata information for other services to discover and interact with.

Server service registry refers to the centralized repository or server that maintains the registry of available services, allowing other services to discover and communicate with them dynamically through a load balancer that has a well-known location.

| Client-Side Service Discovery | Server-Side Service Discovery |
| ------------ | ------------ |
|  **Pros**<br>Client services have control over service discovery and can make dynamic routing decisions.<br><br>Less dependence on external components or infrastructure.<br><br>Client services can easily adapt to changes in service availability. |  **Cons**<br>Centralized service registry provides a single source of truth for service information.<br><br>Improved scalability and performance by offloading service discovery to dedicated components.<br><br>Easy integration with other infrastructure components like load balancers or API gateways.  |
|  **Cons**<br>Increased complexity in client services due to the need for service discovery logic.<br><br>Potential network overhead from querying the service registry for service information.<br><br>Requires additional effort for handling service registry failures or inconsistencies.| **Cons**<br>Dependency on the availability and reliability of the central service registry.<br><br>Possible performance impact due to the additional network hops to the service registry.<br><br>May introduce a single point of failure if the service registry becomes unavailable. |
| ![](https://shaikezam.com/style/client-side-client-registry.png) | ![](https://shaikezam.com/style/server-side-client-registry.png) |
