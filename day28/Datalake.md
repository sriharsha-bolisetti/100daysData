### Data Lake Pattern
- Many companies have more data than they can process it.
- Data Lake is a centralized data repository that can contain structured, semi-strucutured, and unstructured data at any scale.
- Data can be stored in the raw form without any transformations, or some preprocessing can be done before it is consumed.
- From this repository, data can be extracted and consumed to populate dashboards, perform analytics, and drive machine learning pipelines in order to derive insights and enhance decision making.

#### Components of a data lake

1. Loading or Transient Data Zone
2. Raw data zone -> after data quality checks and security transformations have been performed in the transient data zone, the data can be loaded into the raw data zone for permanent storage.
- In raw zone, files are transferred in their native format, `without changing them or associating them with any business logic`,
- All data in the lake should land in the raw zone initially.
- Most users will not have access to the raw zone.
3. Organized or Trusted Data zone
- This is where the data is placed after it’s been checked to comply with all government, industry, and corporate policies. 
- It’s also been checked for quality.
- Serves as `single source of truth` across the data lake for users and downstream systems.
- Data stewards associate business terms with the technical metadata and can apply governance to the data.
- No duplicate records should exist.
- Users have `read only` access in trusted data zone.
4. Curated or Refined data zone
- Data goes through more transformation steps.
- Files may be converted to a common format to facilitate access, and data quality checks are performed.
- Purpose of this process is to prepare the data to be in a format that is more easily consumed and analyzed.
- Transformed data is in this zone.
- Data in this zone can be bucketed into topics, categories, and ontologies.
- Refined data could be used by a broad audience but is not yet fully approved for public consumption across the organization. In other words, users beyond specific security groups may not be allowed to access refined data since it has not yet been validated by all the necessary approvers.

##### Sandboxes
- Sandboxes are an integral part of the data lake because they allow data scientists, analysts, and other users to manipulate, twist, and turn data to suit their individual use cases. 
- Sandboxes are a play area where analysts can make data changes without affecting other users

#### Characteristics of a Data Lake

- Size
- Governability
- Quality
- Usage
- Variety
- Speed
- Stakeholder and Customer Satisfaction
- Security

##### Case Study - GOOGLE

- Data and metadata curation
- Web content homogeneity
- Enhancing document relevance based on document relationships 
- Veracity and Validity of data -> Versioning and Expiration.

#### Locating Documents across the enterprise using faceted search

- Can comprise of
1. Fact retrieval searches
2. More complicated exploratory searches
3. Discover oriented searches suited to problem solving.

- Challenges
1. Security
2. Maturity

#### Data Lake Best Practices

- With help of Machine Learning 
- Amazon Kendra
- Natural Language Processing and Natural Language Understanding.
- Entity Extraction 
- Security

- The Silo Mentality
- Data Governance
- In order to fully trust and track the data in the lake, Context need to be provided to data by instituting policy-driven processes to enable the classification and identification of the ingested data.
- The enormous volume and variability of data in today’s organization complicates the tagging and enrichment of data with the data’s origin, format, lineage, organization, classification, and ownership information.
- Data Governance
- Relavent Metadata
- Data source and lineage -> lineage metadata should be included as part of the ingestion process in an automated manner.
- Physical structure, redundancy checks and job validation.
- Data domain and meaning
- Data governance metadata can be tracked by various ways, some of the recommended ways are
- S3 metadata
- S3 tags
- Enhanced data catalog or vendor to maintain this information.