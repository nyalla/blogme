---
template: blog-post
title: Spring boot application with Kafka, Elastic Search, Redis with enterprise
  standards at different levels of software development [Beginning]
slug: Spring-boot-application-Beginning
date: 2020-07-09 16:02
description: Spring-boot-application-Beginning
---
There are many resources online when we want to learn about different tools for standard enterprise applications. But there are no specific resources which scatter all the enterprise standards. There are no clear tutorials on how the requirement is seen at different levels of software development. I am going to discuss the aspects of the regular enterprise development scenarios and how each person has to responsible for each level of software development.

> This article is mainly for the developers who don't have much
> experience in the software industry. This article is a bit long, but
> covers most of the required scenarios which make you comfortable with
> the software development.

So any software development can be designed by 3 levels.

1.  BA, who takes the requirement from customer
2.  Architect, who designs technical Skelton and usage of sequential diagrams to translate BA descriptive requirements to technical blueprints
3.  Developers, who actually develop the BA requirements to actual working product based on Architect guidelines.

To make the above explanation more clear I will take an example and will walk through the entire process of designing the application.

Firstly will take a use case provided by the BA. Basically BA will give a visual representation of what and how the end product has to be looked with specific use cases.

#### Usecase-1: Page should have a form that takes details of the Customer. Users can enter the details and can submit the details.

#### Usecase-2: Page should display the details of the customer.

  

**Note**: Description of the use cases is much self-explanatory. Note: We are not going to develop the UI in this tutorial. Mostly we will be concerned about the backend, how things will work.

So as per the discussion, the next task is to the Architect who designs the system. So Architect has to think about different aspects like scalability, reliability, etc to build a robust system. At this stage, we will get decisions like what tools we have to use and how the interaction happens at technical stages.

1. microservice architecture - for modular and independent software components

2. Kafka - for event-driven communication between services

3. MySQL - for storing details in the relational way (Point of source)

4. Elastic search - for storing customer details in document way to access data faster

5. Redis - for cache management to accessing customer details faster

6. Spring boot - for developing services very fast and RESTFul way

  

Questions that developers should ask the architect to get a clear understanding.?

**Why Kafka**  - Kafka has been an industry-standard pub-sub model. So we can just post an event and let the consumers decide what to do with that event. Just think of you handing over the control to a pool and different consumers taking that event and do their implementations.

**Why Elastic search**  - ES has been a documented data storage without specific schema in storing the details. For example, for storing customer details in MySQL we might have different tables like a customer, Address, City, Country. When we want all these details it will be a bit of joins and a more time-consuming process. In elastic search, we maintain all these details as one document and internal mechanism in ES maintains indexing, which makes the read access much faster than traditional RDBMS.

**Why Redis** - Redis has been cache management in the system. Just think like a RAM in our system. As we all know RAM data access is faster than Harddisk data.

So BA use cases are now translated to technical use cases to understandable to developer language.

#### Technical Usecase-1 :

1.  Take the JSON input and save it into Mysql.
2.  After a successful save,
    1.  Send response as success to UI.
    2.  Async call, to save data in Elastic search.

#### Technical Usecase-2 :

1.  Take the customer id as input.
2.  Search data in Redis with customer id as a key.
    1.  If data available in Redis, send the response to UI.
    2.  If not available in Redis, query to Elastic search. Then send data to UI.
    3.  Async call, save data in Redis.

Apart from this, we can also get sequential diagrams for each use case from Architect.

#### Sequence diagram for Technical Usecase-1 :

We can see these types of sequence diagrams from Architect to convey the different actions that have to be implemented.

Based on the Architect use cases, now the developer has to design technical document which talks about the ground details about the implementation. The developer has to talk about the methods and services which she/he is developing and how communication happens with different services.

At a very high-level developer will write the use cases as technical documents as below.

#### Module Service 1: Customer Controller

Post - to save customer details

1.  Save into MYsql DB
2.  Post a message into Kafka Topic (should be asynchronous) (which is in Module Service 2)

Get - to get customer details

1.  Check with the given ID data available in Redis ((which is in ModuleService 2 : Redis service Get method ))
    1.  If avail get data and send back
    2.  If not, get data from ES ((which is in Module Service 2: Elastic service Get method )) and send the response and save it to Redis((which is in Module Service 2: Redis service Post method )).

#### Module Service 2: Generic External Service

**Kafka Service**
-   listen to the topic and call ES service method to save

**Elastic search service**
-   save data in the customer document
-   get data from the customer document

#### Redis service
-   save data for the customer id key
-   get data from the customer id

After all, these things are clear and everyone okay with requirements then the actual coding process will kick in.

> See the timeline, how at each level the requirement is represented.
> From a very common descriptive requirement to actual technical
> requirement. So the planning and technical representation are very
> crucial in software development.

Take a break and have a coffee. Relax....

Try to understand the above requirement and try to implement the logic as suggested by technical use cases.

More in-depth technical implementation of how the code flows, I will cover in a separate article.

**Note**: For the installation of all the required tools, there are plenty of tutorials available on the internet. I am not going to list all those in this article. We mainly concentrate on the actual design.
