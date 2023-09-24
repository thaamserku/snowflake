---
created: 2023-09-24T11:56:32 (UTC -04:00)
tags:
  - snowflake
source: https://thinketl.com/what-is-snowflake/
author: ThinkETL
---

# What is Snowflake? 
> [!Excerpt]
> Understand what Snowflake is and how is it different from other data cloud platforms, its key features and benefits.


## **Introduction**

Building traditional data warehouses involved purchasing the required hardware infrastructure and software licenses and maintaining the data in your own data centers. It involves a huge capital expenditure for setting up the datacenters and maintaining the operational overhead is huge. Scalability is another challenge to meet the growing demand.

Developed in 2012 by **Benoit Dageville**, **Thierry Cruanes** and **Marcin Zukowski**, Snowflake is a data platform built on cloud designed to overcome all the challenges of maintaining a traditional data warehouses along with some powerful in-built features for storing, managing and analyzing your huge amounts of data.

_Interesting detail: The name Snowflake was chosen because of the founders’ common love for skiing._

Snowflake is a single, integrated analytic data platform built for cloud delivered as Data warehouse as a service. It is a fully managed SaaS (software as a service) offering that enables data storage, processing, and analytic solutions that are physically separated but logically integrated which are faster, easier to use and far more flexible than traditional offerings.

Snowflake provides a single cloud data platform for Data Warehousing, Data Lakes, Data Engineering, Data Science, Data application development, and secure sharing and consumption of real-time / shared data. 

![](https://thinketl.com/wp-content/uploads/2022/02/66-1-Snowflake.png)

## **How Snowflake is different?**

Snowflake is built on top of the leading cloud computing platforms **Amazon Web Services**, **Microsoft Azure**, and **Google Cloud** infrastructure. It means it runs completely on cloud infrastructure. There is absolutely no hardware or software to select, install, configure, or manage. All the ongoing maintenance, management, upgrades, and tuning are handled by Snowflake.

![](https://thinketl.com/wp-content/uploads/2022/02/66-2-Snowflake-Cloud-Platforms.png)

What sets Snowflake apart is its architecture and data sharing capabilities. Snowflake’s unique architecture consists of three key layers – Database Storage, Query Processing and Cloud Services.

The Snowflake architecture allows storage and compute to scale independently, so customers can use and pay for storage and computation separately. And the sharing functionality makes it easy for organizations to quickly share governed and secure data in real time.

## **Key Features and Benefits of Snowflake**

Snowflake is built specifically for the cloud designed to support fault isolation, performance isolation and elasticity of cloud. Here are key features and benefits of snowflake.

### **Effortless Usability**

As a Snowflake user, you simply signup, choose your cloud provider, load your data into database tables and start querying. There is no hardware (virtual or physical) to select, install, configure, or manage. Snowflake eliminates most of the tuning knobs and parameters required by other data warehouses. As Snowflake is built on top of cloud providers, it is important to choose the cloud provider and its deployment region. However, after choosing your infrastructure provider, you do not interact with the cloud provider and you are not charged by them.

### **Performance and Speed**

The elastic nature of this platform allows you to scale up the compute resources as required instantly to make the data loads and querying faster and scale down afterwards or even shut down when not in use.

The provisioning and deprovisioning of compute resources can be handled directly from Snowflake. So, on the whole, the user need not worry about managing or tuning the underlying cloud clusters to load the data faster or to run a high volume of query. 

### **Storage and support for structured and semi-structured data**

Snowflake provides unlimited scalability in terms of storage and you only pay for what you are using. It also supports both structured and semi-structured non-relational data for loading and analysis it into the cloud database without the need for conversion or transformation into a fixed relational schema. Snowflake automatically optimizes how the data is stored and queried.

### **Data Sharing and replication capabilities**

Snowflake makes it easy to replicate your data across multiple regions and clouds. For companies that need to have their data available in more than one region or on more than one cloud platform, Snowflake provides secure data sharing and replication options that make this easy to achieve.

### **High Availability**

Data in Snowflake is distributed across availability zones of the platform on which it runs — either AWS or Azure or GCP. Availability Zones are replications of data within a region that are used to ensure that data is always available to end users, even if one copy of the data becomes unavailable.

Snowflake maintains data recovery and failsafe in addition to the assurances provided by the cloud provider’s availability zone designs.

### **Third-party data integrations**. 

Snowflake lets you connect with Snowflake customers to extend workflows with data services and third-party applications. An integration platform as a service (iPaaS) like Informatica Cloud [](https://snaplogic.wpengine.com/)pre-built Snowflake connectors make it easy for anyone to create data pipelines to automate workflows across the enterprise.

### **Cost Effective**

Snowflake offers a flexible pricing model where you pay only for the compute and cloud storage that you actually use. The Snowflake interface cuts off idle time of resources and only considers the usage time. They offer multiple pricing options for Snowflake accounts including on-demand per-second pricing with zero long-term commitments or pre-purchased Snowflake capacity options.

## **Conclusion**

Snowflake is a one cloud data platform for many data workloads. Snowflake is built from the ground up for the cloud. Snowflake’s unique multi-cluster shared data architecture delivers the performance, scale, elasticity, and concurrency today’s organizations that simply isn’t possible with a traditional approach.
