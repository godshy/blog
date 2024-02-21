---
title: Cloud native and its implementation in Telco
date: 2023-08-24 13:48:22
tags: Cloud native
category: Learning
---
# Cloud native in telecom 

**Right now it is unclear what is the concept of Cloud Native and what is its implementations, so main goal of this blog is to simply introduce the basic concept and how Cloud Native is composed.**
## Index
1. [What is Cloud Native？](#what-is-cloud-native)
2. [Basic principles and components of Cloud Native](#basic-principles-and-conponents-of-cloud-native)
3. [Cloud Native design principles](#cloud-native-design-priciples)
    1.  [Agnosticism](#agnosticism)
    2.  [Software decomposition and Life Cycle Management](#software-decomposition-and-life-cycle-management)
4. [Some Q&As](#qas)



## What is Cloud Native？
Cloud native is not a particular product or technology. Instead, it's a principle, a business pattern. Cloud Native is a term that describes the patterns of organizations, architectures and technologies that consistently, reliably and at scale fully takes advantage of the possibilities of the cloud to support cloud-oriented business models. It is the combination of different kinds of best practices.Enterprise and developers use technologies based on Cloud Native such as CI/CD, container and microservices to achieve high availability, high reliability scalability and lower Time To Market and delivery.


## Basic principles and components of Cloud Native
What can Cloud Native bring us?
- Speed: Fast introduction of new services
- Scale: Easily expand from hundreds of users to millions
- Performance: Optimized capacity throughput and resource utilization
- Efficiency : Low cost for both Speed and Scale. Automation, zero-touch operations, legacy and new services, life-cycle independence for both Services and
Infrastructure

Therefore, target of Cloud Native is to achieve higher TTM(Time to) and lower TCO(Total Cost of Ownership) without sacrificing service reliability and performance. Let's see what are the basic components of Cloud Native.

- Cloud Native Applications
- Microservice
- APIs
- Containers and container orchestration
- Other automating tools
- Cloud native friendly infrastructures(Public clouds，private clouds，mixture clouds)

Like I mentioned earlier, cloud native doesn't only contain technologies, but also idea and behavior patterns. Let's see what needs to be considered in mind about cloud native.

- CI/CD (Continuous delivery and continuous integration)
- Highly automated (includes CI/CD，along with auto monitoring，data handling and analysis with ability to anomaly detection)
- Embrace actively to open source and collaboration with CNCF (cloud native computing foundation): CNCF is an open source Cloud Native organization, it is responsible to develop and maintain crucial parts close related to Cloud Native. Please refer to their [website](https://www.cncf.io/).

To sum-up, a company need to achieve the following to completely take the advantage of cloud native.

1. Posses complete infrastructure and environments for cloud native applications
2. Have the ideal of automation on duplicated process
3. Contribute to open source society, continuous delivery of value to the user

![Cloud native requires both infrastructure and development patterns to reach the target.](https://s2.loli.net/2023/09/14/2uHknclWej18IiB.png)

## Cloud Native design principles
Let's talk about the design principles of cloud native application. Since it is one of the most important parts in the whole could native concept and it is rather new in the industry.  Graph below describes the basic principles need to considered before deploying and designing cloud native applications.

![Cloud Native Application Design Principles](https://s2.loli.net/2024/02/21/s6AUd5MEwOl9IDV.png)

### Agnosticism

Cloud native applications need to to be agnostic to the underlying cloud infrastructure. Although this does not necessarily apply to the traditional Cloud Native players like Netflix and Uber since they control the full stack that they are deploying their application on.
As we evolve the cloud infrastructure towards supporting container infrastructure (CaaS) the agnosticism requirements remain but the challenges are different. A CNA requires a CaaS and must be able to be deployed on whatever CaaS and IaaS combination that an operator has in their cloud environment, see figure below for CNA agnosticism scenario. A CNA must be able to run on any modern kernel without requiring any proprietary additions. Optional use of plug-ins for optimizing performance is allowed.

![Agnosticism explained](https://s2.loli.net/2023/10/18/ctJGAWqylKeRxuz.png)


### Software decomposition and Life Cycle Management

Another important principle is for CNA itself, which requires CNA to be decomposed into smaller, more manageable pieces. This is usually done through utilizing a microservice architecture (MSA). MSA is a design pattern that strives to structure an application as a collection of loosely coupled stateless services.
![Microservice architecture](https://s2.loli.net/2023/10/23/KhxQlkCBGo7WI9z.png)
This concept is not new in only for cloud native, dividing code into more manageable pieces and call them "code modules" or "software components" has always been a good practice in software development. However, in cloud native realm, those software components, AKA microservices, now have a well bounded scope and can be individually deploys, scaled and upgrade using CaaS environment. In addition, microservices have their own defines and version-controlled network-based interfaces to communicate with each other, which enables the lifecycle management for each microservice.

#### Scaling
#### Upgrade
## Q&As