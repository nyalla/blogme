---
template: blog-post
title: "Conduktor : Kafka UI "
slug: kafka-ui
date: 2021-05-02 15:06
description: kafka, conduktor,kafkaui,ui,productivity
---
Recently I had introduced to the tool called Conduktor., which gives UI to kafka operation. :) 

As a developer usually interacting with kafka not that much simple. To post a messsage and consume a message via command line bit a hectic process.

Lets start working on Conduktor. 

Firstly we have to install windows desktop application from conduktor website. https://www.conduktor.io/download/

Make sure that your kakfa cluster is working on the machine you wanted to connect the **Conduktor.**

To test whether Kafka cluster is up and able to access. Run below command

kafka-topics --list --bootstrap-server localhost:9092



After installation we need to sign into the Conduktor license (provided free plan with single broker). Then open Conduktor app from the list.



**For local testing:**

![conducktor connection](/assets/conduktor_con.png "conducktor connection")



Click on test Kafka Connectivity to check the connection. If everything went well you will see the screen as below.



![conduktor home](/assets/conduktor_home.png "conduktor home")



All the operations like  producing , consuming and adding topics we can directly do from the UI.



Happy explore...