## Terms
- Data Intensive: if data management is one of the primary challenges in developing the application $\rightarrow$ storing & processing large data volumes, managing changes to data, ensuring consistency in the face of failures and concurrency, making services highly available
- Compute Intensive: if parallelizing a very large computation is one of the primary challenges
- Transaction: group of reads and write that form a logical unit
- Online Transaction Processing (OLTP): A class of software systems designed to manage, execute, and record high volumes of daily, operational transactions in real time. It is optimized for fast, reliable, and concurrent processing of small, simple database queries—such as inserts, updates, and deletes—across multi-user environments.
- Online Analytical Processing (OLAP): a category of software technology that enables analysts, managers, and executives to analyze multidimensional data interactively and rapidly from multiple perspectives. While operational databases focus on fast transaction recording, OLAP systems are optimized for complex, read-heavy analytical queries over massive historical datasets to drive business intelligence (BI), decision-making, and reporting.
- Point Query: A database query that targets and retrieves a single specific row or record from a table, typically by searching for an exact match on a unique identifier such as a Primary Key or a unique indexed column.
- Data Silos: Isolated islands of data controlled by a single department or business unit, inaccessible to the rest of the organization. They cause duplicate records, inconsistent metrics, and prevent a unified view of business operations.
- Data Warehouse: A centralized, highly structured relational database optimized for fast analytical queries and business intelligence (BI). Data is cleaned, transformed, and organized into specific schema (like star schemas) before storage, making it ideal for structured operational data and standardized reporting
- Data Lake: A centralized repository that stores vast amounts of raw data in its native format—including structured, semi-structured (JSON, XML), and unstructured (logs, images, audio) data. It allows organizations to dump data at scale immediately and apply structure later when querying, making it ideal for data science and machine learning.
- Extract-Transform-Load (ETL) / Extract-Load-Transform (ELT) Pipelines: Automated workflows that move data from source systems to a target destination:
	- **ETL:** Extracts raw data, transforms (cleans/aggregates) it on a separate processing server, and loads the transformed result into a database or warehouse.
	- **ELT:** Extracts raw data and loads it directly into the target system (like a modern data lake or lakehouse) first, leveraging the destination's powerful compute engines to transform the data on demand.
- Cloud Native: An architectural approach to building and running applications that fully leverages the cloud computing model. It relies on microservices, containerization (e.g., Docker, Kubernetes), serverless design, dynamic scaling, and managed services rather than simply lifting and shifting legacy virtual machines into the cloud.
- Distributed System: A collection of independent physical or virtual machines (nodes) connected via a network that communicate and coordinate actions by passing messages. To the end user, the network operates as a single, coherent system, providing horizontal scaling, high availability, and fault tolerance.
- Node: A single computational unit—such as a physical server, virtual machine, or container—within a distributed system or network. Each node possesses its own CPU, memory, and storage, and executes tasks independently as part of the broader system.
## Common Building Blocks
- Database
- Cache
- Search Index
- Stream Processing
- Batch Processing

## Operational Vs Analytical Systems
- Teams
	- Backend Engineers
	- Business Analysts (Business Intelligence) / Data Scientists
		- actions
			- perform analytics
			- typically do not modify the data (though they might create derived datasets in which the original data has been processed in some way)
- Systems
	- Operational Systems:
		- Backend Services & Data Infrastructure where data is read, created, and modified in a database based on actions performed by users
	- Analytical Systems
		- serve the needs of business analysts and data scientists
- Roles
	- Data Engineers
		- people who know how to integrate the operational and analytical systems and who takes responsibility for the organization's data infrastructure 
	- Analytics Engineers
		- model and transform data to make it more useful for business analysts and data scientists in an organization
### Transaction Processing & Analytics
- Online Transaction Processing (OLTP)
	- an Operational System looks up a small number of records by a key (called a _point query_) $\rightarrow$ records are inserted, updated, deleted based on the user's input
	- mostly run a fixed set of queries that are baked into the application code with one off custom queries used occasionally for maintenance or troubleshooting  
- Online Analytical Processing (OLAP)
	- scans over a huge number of records and calculates aggregate statistics (count, sum, avg, etc) rather than returning the individual records to the user
		- EX: what was the total revenue of each of our stores in January
		- EX: how many more bananas than usual did we sell during out latest promotion
		- EX: Which brand of baby food is most often purchased together with brand X diapers
	- gives users the freedom to write arbitrary queries by hand or to generate queries automatically by using data visualization or dashboard tools such as Tableau, Looker, or Microsoft Power BI
	- Product Analytics / Real-Time Analytics
		- designed for analyticsl workloads and embedded into user-facing products $\rightarrow$ Pinot, Druid, ClickHouse $\rightarrow$ ingest data in real time and are optimized for low-latency query responses vs OLAP systems that typically ingest data in batches and are optimized for high-throughput query processing
- OLTP vs OLAP

| Property            | Operational Systems (OLTP)                      | Analytical System (OLAP)                  |
| ------------------- | ----------------------------------------------- | ----------------------------------------- |
| Main Read Pattern   | Point Queries (fetch individual records by key) | aggregate over large number of records    |
| Main Write Pattern  | create, update, and delete individual records   | bulk import (ETL / ELT) or event stream   |
| Human User Example  | end user of web/mobile application              | internal analyst for decision support     |
| Machine Use Example | checking if an action is authorized             | detecting fraud / abuse patterns          |
| Type of Queries     | fixed, predefined by application                | arbitrary, ad-hoc exploration by analysts |
| Query Volume        | lots of small queries                           | few queries, each is complex              |
| Data Represents     | latest state of data (current point in time)    | history of events that happens over time  |
| Dataset Size        | gigabytes (GB) to terabytes (TB)                | terabytes (TB) to petabytes (PB)          |
- Some system offer _hybrid transactional/analytical processing (HTAP)_ which aims to enable OLTP and analytics in a single system without requiring ETL from one system into another
	- these often still have the same OLTP vs analytics system + shared interface hidden behind the scenes
	- does not replace a data warehouse 
### Data Warehousing
- strategy used to separate the analytics workload from the OLTP workload
- it is generally undesirable for business analysts and data scientists to query OLTP systems directly
	- the data of interest can be spread across multiple operational systems making it hard to combine datasets into a single query (a problem known as _data silos_)
	- the kinds of schemas and data layouts that are good for OLTP are less well suited for analytics 
	- analytical queries can be very expensive and running them over OLTP data base can impact performance of other users
	- OLTP systems might be in separate network that users are not allowed to directly access for security or compliance reasons
- a _data warehouse_ is a separate database that analysts can query without affecting OLTP operations 
- contains a real-only copy of the data from all various OLTP systems 
- ETL / ELT Pipelines $\rightarrow$ Data is extracted from OLTP database, transformed into analysis friendly schema, cleaned up, and then loaded into data warehouse 

#### Data Warehouse vs Data Lake
- Data Warehouses often use a _relational_ data model that is queried through SQL $\rightarrow$ good for business analysts but bad for data scientists
	- bad: transforming data into a form that is suitable for training ML models (Feature / Feature Engineering)
	- bad: using natural language processing (NLP) techniques on textual data to try to extract structured information from it 
	- bad: extracting structured information from photos by using computer vision techniques
- Data scientists prefer not to work in relational database such as data warehouse $\rightarrow$ instead they prefer to use Python data analysis libraries such as Pandas  scikit-learn, statistical analysis languages like R, and distributed analytics frameworks such as Spark
	- Data Lake: a centralized data repository that holds a copy of any data that ight be useful for analysis, obtained from operational systems via ETL process
		- a data lake simply contains files
			- without imposing any particular file format (Avro / Parquet / etc), data model / schema
			- can equally contain text, images, videos, sensor readings, sparse matrices, feature vectors, genome sequences, or any other kind of data
		- contains data in the "raw" form produced by the operational systems without the transformation into a relational data warehouse schema 
			- each consumer can transform the raw data into the form that best suits their needs (called the _sushi principle $\rightarrow$ 'Raw data is better'_)
- ETL Pipelines can use a Data Lake as an intermediate stop on the path from the operational system to the data warehouse 
### Systems of Record (_source of truth_) and Derived Data (_redundant data_)
- System of Record
	- holds authoritative of canonical version of the data. If there is any discrepancy between another system and the system of record the system of record (by definition) is the correct one
- Derived Data Systems
	- is the result of taking existing data from another system and transforming or processing it in some way
	- can be recreated from its original source if lost 
	- EX: cache $\rightarrow$ data can be served form a cache if present but if the cache does not contain what you need you can fallback tot he underlying database
	- while _redundant_ it is often essential for getting good performance 
	- analytical systems are usually derived data systems because they are consumers of data created elsewhere

## Cloud Versus Self-Hosting
- "should you build it or buy it"
	- middle ground = off-the-shelf software (open source or commercial) that you _self-host_ or deploy yourself 
- related question is _how_ you deploy services $\rightarrow$ cloud || on premises 
### Pros and Cons of Cloud Services
- Yourself / On Premises / Self Host
	- if you have experience setting up and operating the system you need and load is predictable $\rightarrow$ it is often cheaper to buy your own machines and run the software on them yourself
	- can configure and tune the system to your specific workflow
- Cloud Services
	- if you need a system that you don't already know how to deploy and operate then adopting cloud service is often easier and quicker than learning to manage the system yourself
	- generally have a broadly efficient system due to the SaaS provider implementing broad operational expertise from serving many customers workloads
	- very valuable if your load varies alot over time $\rightarrow$ scale up and down when needed and thus you are not paying for peak load infrastructure that you are not using most of the time
	- Biggest Downside $\rightarrow$ you have no "control" over it 
		- lacking a feature $\rightarrow$ politely ask the vendor and they may or may not add it as some unknown time in the future
		- services go down $\rightarrow$ you can only wait for them to come back up
		- hard to debug 
		- vendor lock in $\rightarrow$ if the service shuts down or become expensive you are at their mercy 
		- vendor needs to be trusted to keep data secure which can complicate the process of complying with privacy and security regulations
### Cloud Native System Architecture
- in principle any software that you can self-host could also be provided as a cloud service however systems that have been designed from the ground up to be coud native have been shown to have several advantages
	- better performance on same hardware
	- faster recovery from failures
	- quickly scale computing resources to match load
	- supporting larger datasets 

| Category         | Self-Hosted Systems         | Cloud Native Systems                                      |
| ---------------- | --------------------------- | --------------------------------------------------------- |
| Operational/OLTP | MySQL, PostgreSQL, MongoDB  | AWS Aurora, Azure SQL Db Hyperscale, Google Cloud Spanner |
| Analytical/OLAP  | Teradata, ClickHouse, Spark | Snowflake, Google BigQuery, Azure Synapse Analytics       |
#### Layering Cloud Services
- a key idea of cloud native services is no tonly to use the computing resources managed by your operating system but also to build upon lower-level cloud services to create higher-level services
	- Object storage services (Amazon S3, Azure Blob Storage, Cloudflare R2, etc...) provide more limited APIs than a typical file system (basic file read & write) but they have the advantage of hiding the underlying physical machines $\rightarrow$ the service automatically distributes the data across many machines so you dont have to worry about running out of disk space on any one machine or if some machines / disks fail no data is lost
	- 
#### Separation of Storage and Compute
- RAID (Redundant Array of Independent Disks): used to maintain copies of the data on several disks attached to the same machine
	- can be implemented in hardware or software by the operating system and is transparent to the application accessing the filesystem
- Local Disk vs Virtual Disk
	- Local Disk $\rightarrow$ cloud systems treat them as ephemeral caches and less like long term storage because local disks are inaccessible if the associated instance fails or if the instance is replaced with a bigger or smaller instance 
	- Virtual Disk $\rightarrow$ can be detached from one instance and attached to another one 
		- not a physical disk but rather a cloud service provided by a separate set of machines that emulates the behavior of a disk 
- Traditional Systems Architecture $\rightarrow$ same computer for both storage (disk) and computation (CPU & RAM)
- Cloud Native Systems $\rightarrow$ storage and computation are separated (_disaggregated_)
	- EX: S3 only stores files $\rightarrow$ if you want to analyze that data you have to run the analysis code somewhere outside S3 

### Operations in the Cloud Era
- Traditional: _Database Administrators (DBAs)_ & _System Administrators (sysadmins)_
- Recent: integrate roles of software development and operations into teams with a shared responsibility for both backend services and data infrastructure (_DevOps_)
	- Setting up automation $\rightarrow$ repeatable process over manual one-off jobs
	- using ephemeral VMs and services rather than long running servers
	- enabling frequent application updates
	- learning from incidents
	- preserving the organizations knowledge about the system even as individual people come and go
- "Operations"
	- ensure services are reliably delivered to users (including configuring infrastructure and deploying applications) and to ensure stable production environment (including monitoring and diagnosing any problems that may affect reliability)
	- emphasis has shifted from individual machines to services 
- Role Bifurcation
	- Operations teams as infrastructure companies specialize in the details of providing reliable service to a large number of customers while the customers of the service spend as little time and effort as possible on infrastructure
		- customers of cloud services still require operations but that focus on different aspects
			- Capacity Planning becomes financial planning
			- Performance Optimization becomes cost optimization 

## Distributed Versus Single-Node System
- _distributed system_ = a system that involves several machines communicating via a network
- _node_ = each process participating in a distributed system
### Reasons to use Distributed System
- Inherent Distribution: 
	- if an application involves two or mote interacting users, each using their own device then the system is unavoidably distributed $\rightarrow$ communication between the devices will have to occur via a network
- Requests between cloud services
	- data stored in one service is processed in another that data must be transferred  over the network $\rightarrow$ cloud native & microservices are therefore distributed
- Fault tolerance / high availability
	- application needs to continue working even if one machine (or several) goes down
- Scalability
	- data volume or computing requirements grow bigger than a single machine can handle you can spread the load across multiple machines
- Latency
	- users around the world $\rightarrow$ might want servers in various regions worldwide to avoid slow network packets travel across the world 
- Elasticity
	- application is busy as some times and idle at others $\rightarrow$ cloud deployment could scale up or down to meet demand. Hard to do on single machine that usually need to be provisioned to handle maximum load  
- Specialized Hardware
	- different parts of the system can take advantage of different types of hardware to match their workload
- Legal Compliance
	- data residency laws 
- Sustainability
	- flexibility in when / when to run jobs that might run them at off times with plenty of renewable energy
### Problems with Distributed Systems
- every network request needs to deal with the possibility of failure
- hard to troubleshoot
- not optimized for all workloads $\rightarrow$ in some cases a single-threaded program on one computer can perform better than a cluster over 100 CPU cores
### Microservices and Serverless
- Service Oriented Architecture (SOA)
	- divide the system into clients and servers and let client make requests to the servers
- _Microservices_ = Refinement of SOA
	- service has one well-defined purpose & exposes an API that can be called by clients / servers on the network 
	- each service has a single team responsible for its maintenance
	- advantages
		- each service can be updated independently
		- hiding implementation details behind an API allows the service owners to change the implementation without affecting clients
		- services have their own databases
	- disadvantages
		- breeds complexity 
		- complicated testing
		- each service requires infrastructure for deploying new releases, allocating hardware resources to match load, collecting logs, monitoring health, etc
			- Kubernetes
		- APIs can be challenging to evolve 
			- OpenAPI, gRPC 
		- a technical solution to a people problem 
			- allowing different teams to make progress independently wihtout having to coordinate with eachother 
- _Serverless (Function as a Service)_
	- cloud provider automatically allocated and frees hardware resources as needed based on the incoming requests 
	- just like cloud storage replaced capacity planning with metered billing $\rightarrow$ serverless is bringing metered billing to code execution: you pay for the time that your application is running rather than having to provision resources in advance
### Cloud Computing VS Supercomputing (_High-Performance Computing (HPC)_)
- used for computationally intensive scientific computing tasks
- typically runs large batch jobs that if they fail can be entierly stopped, fixed, and restarted (uptime is less important)
- noes communicate through shared memory and RDMA for high bandwidth and low latench but assume high level trust among the users




