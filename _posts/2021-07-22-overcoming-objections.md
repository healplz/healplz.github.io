# Overcoming Objections

Recently I was presented with an exciting challenge: provide answers for a list of objections to data warehousing. Below I will attempt to address these objections out of the hope that someone will find my arguments helpful.

## Objections

### Data warehousing adds too much complexity to the data model

Data warehousing is complex. I certainly can't claim to know everything about it. I do not view the complexity as a bane, however, but a boon. The data warehousing methodology we employ, Data Vault 2.0, has a relatively simple set of rules at its core. While at first, it can seem overwhelming, I do not believe that it is needlessly complex. The complexity exists to enable to most flexibility. There is no need to make massive transformations on the incoming data to store it. Instead, it is partitioned out in fairly logical segments to keep growth under control -- storing changed records, not duplicating key values, storing relationships in their respective link tables, etc.

### Data warehousing requires too much storage space

I will admit that Data Vaults can be quite large. That goes with the territory when you are storing historical data for any decently large relational database system. Storage is getting cheaper constantly; with Snowflake, most of the cost is for computing resources rather than storage. Also, materializing views of the data adds more storage overhead, but this gets back to a classic software tradeoff: speed vs. storage. Setting aside the time and storage to materialize the most recent point-in-time versions of the warehoused objects enables them to quickly return results without sacrificing the historicity or structure of the underlying warehouse.

### You have to load all of the columns, even ones you don't need

This one left me stumped at first, but they went on to explain: a large amount of business logic went into informing the previous warehouse design, so they felt that the warehouse layer was the source for their downstream reports. In our warehouse, those requirements did not inform loading of the raw vault. The design goal for the raw vault is to leave the warehouse as broad as possible, preferring to have business rules in the analytics layer. Long term, we believe this allows the most flexibility when responding to evolving business needs.

### You have to load and compare all of the data every day

Unfortunately, if your source system does not publish change records, you will have to publish every record to the warehouse and check for changes. We hash the data columns and compare them to the most recent stored version to facilitate these comparisons. This hash-based comparison of records has proved to be a relatively quick way to make these checks. Also, these checks serve to keep the raw vault from growing linearly over time (scaling by the full size of the source systems daily).

### Even though the data in these objects is the same, you require multiple tables to store them

This one is relatively niche and may require some context. The source system in question was structured as several source systems. While the product was truly the same in each instance, each region had its own installation. And each installation had a different schema. To account for this, the analytics layer presents a merged version of the data from the source systems (and handling duplicate keys where encountered), but the underlying raw vault stores the data separately. Fundamentally, storing data from systems with different underlying structures mixed would violate the integrity of the data -- hiding possible valid nulls and introducing possibly invalid nulls at the record level. Merging them at the record label would also introduce issues tracking the historical changes of the source system and interfering with tracking data lineage.

## Warehousing vs. Analytics

We zeroed in on their objections being rooted in their understanding of data warehousing. To them, there is no distinction between the warehouse and analytics layers. In their previous implementation, there was also no distinction. Business logic informed the structure and loading of data marts directly, and that collection of marts was called the Data Warehouse. To mitigate this, it is crucial to listen to the objections non-judgementally. Seeking mutual understanding should be the goal of any such encounter. While, in the end, we may not push to deliver a data warehouse as they understand it, our ultimate goal remains to provide them a solution to meet their analytics and storage needs.