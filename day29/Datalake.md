### Data Lake Architecture: Data Lake vs Data Warehouse in Modern Data Management

Demands of the digital era

- Granular personaliztion, individual treatment.
- Contextual insight
- Actionable insight
- Smar, analytics enabled services
- Ability to adopt changes in business landscapes.

New Data Volumes
- Information sources
- Information pipes
- Data volumes

New Data is powering new business oppurtunities.
 Traditional Data Analytics Architecture
 - Implementation cycles of analytics take too long.
 - Siloed departmental data.
 - Lack of support of unstructured data.
 - Lack of support for ad-hoc analytical queries.
 - Poor or non-existent metadata management and data lineage.

#### Data Lake
- System or repository of data stored in its natural/raw format, usually object blobs or files.
- Quicky ingest anything  - do not enforce schema on write
- All data in one place
- Low cost scalable storage, decoupled from compute (in cloud).
- Driver anywhere, flexible access, schema-on-read -> diverse toolset and processing types based on data and use case.
- Future proof.

##### Data Lake User Needs are Different Too
- Business Users
- Business Analysts
- Data Scientists, Engineers and App Developers.

##### How Data Lakes are Organized

- Data Lake Core components

- Source Anything
- Ingest anything without etl
- Store any data, any volume
- Catalog & Discover
- Batch processing
- Stream Processing
- ML & Analytics

###### Data Ingestion
- Enterprise Sources
1. Data workload migration
2. Ingestion in batches
3. Incremental Ingestion

- External Sources
1. Streaming Data Ingestion.

Cold Path & Hot Path

Streaming Ingestion if 
- Log files
- Ingest applicaiton events

CDC

- Continues ingestion in data lake
- Capturing streaming changes
- Database migration to cloud

###### Storage
Zones

- Raw -> Immutable store that cannot/should not be changed after it has been written. Self descriptive with metadata. 
- Optimized
- Analytics
- Quarantine

- File Formats

Partitioning

- Raw
- Optimized Analytics

Data Catalog 
- Data governance and security.
- Crawlers.

Cost Optimization

- Retention policies

###### Analytics

- Interactive Analytics
- Big Data Analytics
- Real Time Analytics

#### Data Processing Concepts
- Lambda Lake
- Kappa Lake
- Delta Lake - Unified Batch and Streaming.

#### BI and Visualization Frameworks
- Apache Superset
