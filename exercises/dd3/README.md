# Deep Dive 3 - Replicating data using Replication Flows in SAP Data Intelligence

Technically you can realize data replication use cases using SAP Data Intelligence pipelines, but there are also some important considerations and limitations to mention when using pipelines for such a scenario.<br><br>

![](images/3-001.JPG)
<br><br>

Important aspects to highlight are:
- 1 data set replication ≙  1 pipeline in Data Intelligence with possibility to generalize pipeline execution with variables in certain scenarios

- Recovery of data integration pipelines in case of various error situations using resilience & snapshot functionality in generation 2 pipelines

- High total cost of ownership (TCO) when having large amount of data sets (e.g. hundreds or thousands of CDS Views or tables) in a replication use case which results in creating as well as maintaining a lot of Daa Intelligence pipelines

- Limited performance scalability in pipelines, e.g. in the area of parallelizing initial as well as delta load processes
<br><br>

### **Overview of Replication Flow core functionality**

Therefore, replication flows provisioned via the so called "Replication Management Service (RMS)" have been made available to simplify the realization of data replication use cases in SAP Data Intelligence Cloud. Whereas RMS includes the whole data replication service incl. its dependent components, a "Replication Flow" is the name of the artefact that a user is creating & maintaining inside the SAP Data Intelligence Cloud Modeler application. The main capabilities and functional foundation are visualized in the following illustration:
<br><br>
![](images/3-002.JPG)
<br>

The main functionalities of Replication Flows cover:

- Model data replication from a selected source to a selected target. In this case a more simplified way of realizing "mass data replication use cases" is being offerent to move data very easy from a source to a target system


- Initial focus on 1:1 replication of with simple projections and filters, e.g. adding, adjusting and removal of columns as well as ability to provide row-level filters on one or multiple columns

- Dedicated user interface for modeling mass data replication via a new interface that is embedded in the existing modeler application and optimized for mass data replication scenarios to offer a simplified user experience

- Lower total cost of ownership (TCO) and total development costs (TDC) for customers realizing mass data replication scenarios in SAP Data Intelligence Cloud compared to using pipelines for such use cases.

- Support initial load as well as delta load capabilities, which is based on trigger-based change-data-capture (CDC) using logging tables on the connected source systems

- Support parallelization during initial load through partitioning

- Support resiliency functinoalities & automated recovery in case of error scenarios

<br>

### **Overview of Replication Flow Connectivity**

Looking at the supported source & target connectivty, different SAP and non-SAP connectivty can currently be used when creating a Replication Flow, which can also be checked in our product documentation under the following Link.

**[Replication Flow source and target connectivity ](https://blogs.sap.com/2019/12/16/https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/ca509b7635484070a655738be408da63/f4327d3e2f7146a19e76924f8a79454a.html)** 


The supported source connectivity includes:

- SAP S/4HANA Cloud
- SAP S/4HANA on-Premise
- SAP Business Suite & SAP S/4HANA Foundation via SLT
- SAP Business Warehouse 
- Azure MS SQL

The supported target connectivity includes:

- SAP HANA Cloud
- SAP HANA Data Lake Files (HDL-Files)
- Amazon S3
- Microsoft Azure Data Lake Gen 2
- Google Cloud Storage 
- Kafka

There are partially special configurations available for specific target connections, such as different file formats for target objects stores (e.g. CSV, Parquet etc.) as well as data format & compression for Kafka as a target. More information about these configuration settings can be found in our product documentation. 

**[Connectivity configuration parameters ](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/1c1341f6911f4da5a35b191b40b426c8/a425e3426b644b1184207d291a856119.html)**

<br>

![](images/3-003.JPG)

<br>

### **Overview of ABAP Integration with Replication Flows** 

The folloowing sub-chapter describes a deep dive into the topic how a user can integrate the various types of SAP ABAP based systems as a source with Replication Flows. 

We will tsart with a first high level overview which kind of data sets & artefacts can be integrated with each SAP ABAP system.
<br><br>

![](images/3-007.JPG)

<br>

Now we take this overview and provide some more greanular view on the type of SAP System that can be integration with Replication Flows incl. a brief overview on the minimum version that is required. More information about the ABAP integration with SAP Data Intelligence can be found here: 
**[SAP Data Intelligence ABAP Integration ](https://launchpad.support.sap.com/#/notes/2890171)**

<br>

![](images/3-006.JPG)

<br>

**Important Note:** It is always recommended to check the central SAP Note mentioned above as well as the individual SAP Note we have for each SAP ABAP System for the minimum pre-requisites as well as implementing all referenced SAP notes to fix known issues, e.g.for integrating SAP S/4HANA 2021 you can check the following  Note **[SAP Data Intelligence ABAP Integration - SAP S/4HANA 2021 ](https://launchpad.support.sap.com/#/notes/3085579)**
and for DMIS 2018 SP07 you can check this SAP Note **[SAP Data Intelligence ABAP Integration - DMIS 2018 SP07 ](https://launchpad.support.sap.com/#/notes/2890171)**.

<br>

### **Details on Replication Flow architecture**

In the underlying architecture Replication Flows are executed by so called "worker graphs", which internally built based on Data intelligence pipelines, but optimized for data replication use cases to overcome the limitations we have seen in the beginning of this deep dive when using regular pipelines. <br>

A worker graph is being executed in the background in case a user triggers the execution of a Replication Flow and mainly consists of the source & target connectivity + projection & mapping in case the user is defining a filter or changes the strcuture of the data set. Theoretically, there is no limit for a user to define how much data sets (also known as Tasks) can be added inside a single Replication Flow, but there are some important aspects we will highlight below that influences this decision.<br>

**Important Note:** A user cannot create such a Replication Flow on her/his own and should only be seen as the technical runtime artefact, which is automatically triggered when a Replication Flow is being executed. <br>
Please check below how a worker graph looks like: <br>
<br>

![](images/3-005.JPG)

<br>

In contrast to pipelines, a single worker graph as illustrated above can replicate multiple data sets from the source to the target. Each worker graph has by default a total 10 connections (5 source & and 5 target connections) through which the data can be replicated and by default a single Replication Flow has two worker graphs assigned. This setting can be adjusted so that multiple worker graphs are started for a Replication Flow depending on the use case. Additional information can also be found under the following link:
**[Sizing Replications ](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/ea95bb6d8ac24cd6a4ad396ca5e35bc6/00ce7f17afcb40a287c1946b9abbafbe.html)**


The number of connections per replication flows can be figured in the monitoring application on tab "Replications" on a Replication Flow level using the configuration button. More information can be found here when looking for the actions in the "Replications" tab in the
**[Monitoring application ](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/ca509b7635484070a655738be408da63/e352e7f1a99e4b5d989db5ae1e5e5d0b.html)**


<br>

### **Important Links**

**[TechEd exercises for Replication Flows](https://github.com/SAP-samples/teched2022-DA281/tree/main/exercises/ex3)**
<br>

**[Step by Step Guide for creating a Replication Flow](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/1c1341f6911f4da5a35b191b40b426c8/d3acc43c77c848b6a82d899ff6895f99.html)**
<br>

**[Overview of supported source & target connections](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/ca509b7635484070a655738be408da63/f4327d3e2f7146a19e76924f8a79454a.html)**
<br>

**[Questions? SAP Community](https://community.sap.com/)**
<br>

**[SAP Data Intelligence ABAP User Guide](https://help.sap.com/docs/SAP_DATA_INTELLIGENCE/3a65df0ce7cd40d3a61225b7d3c86703/8b287f0f0033447c8a57a1bee74cd840.html)**
<br>

**[SAP Data Intelligence ABAP Integration - Central SAP Note](https://launchpad.support.sap.com/#/notes/2890171)**
<br>


