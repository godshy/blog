---
title: Cloud native and its implementation in Telco
date: 2023-08-24 13:48:22
tags: Cloud native
category: knowledge sharing
---
# Cloud native in telecom 

**Right now it is unclear what is the concept of Cloud Native and what is its implementations, so main goal of this blog is to simply introduce the basic concept and how Cloud Native is conposed.**
## Index
1. [What is Cloud Native？](#what-is-cloud-native)
2. [Basic principles and components of Cloud Native](#basic-principles-and-conponents-of-cloud-native)
3. [Cloud Native design principles](#cloud-native-design-priciples)
4. [Some Q&As](#qas)



## What is Cloud Native？
Cloud native is not a particular product or technology. Instead, it's a principle, a business pattern. Cloud Native is a term that describes the patterns of organizations, architectures and technologies that consistently, reliably and at scale fully takes advantage of the possibilities of the cloud to support cloud-oriented business models. It is the combination of severals of best practices.Enterprise and developers use thchnologies based on Cloud Native such as CI/CD, container and microservices to achieve high availability, high reliability scalibility and lower Time To Market and delivery.


## Basic principles and conponents of Cloud Native
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

- CI/CD (Continous delivery and continous integration)
- Highly automated (includes CI/CD，along with auto monitoring，data handeling and analysis with ability to anomoly detection)
- Embrace actively to open source and collaboration with CNCF (cloud native computing foundation): CNCF is an open source Cloud Native organization, it is responsible to develop and matain crutial parts close related to Cloud Native. Ple refer to their [website](https://www.cncf.io/).

To sum-up, a company need to achieve the following to compeletely take the advantage of cloud native.

1. Posess complete infrastructure and environments for cloud native applications
2. Have the ideal of automation on duplicated process
3. Contribute to open source society, continous delivery of value to the user

![Cloud native requires both infrastruture and development patterns to reach the target.](https://s2.loli.net/2023/09/14/2uHknclWej18IiB.png)

## Cloud Native design principles
Let's talk about the design principles of cloud native applications. Since it is one of the most important parts in the whole could native concept and it is rather new in the industry.  Graph below describes the basic principles need to considered before deploying and designing cloud native applications.


## Q&As