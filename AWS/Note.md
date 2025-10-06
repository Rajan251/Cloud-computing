**The data transfer will be chargeable**, not free.[1][2][3]

### Cross-AZ Data Transfer Costs

When your application server in AZ 1a communicates with the database in AZ 1b, AWS charges **$0.01 per GB** for data transfer in each direction. This means you pay **$0.02 per GB total** for bidirectional traffic—$0.01 for data going from the application to the database, and another $0.01 for data coming back from the database to the application.[2][3]

### Why It's Not Free

Data transfer is only free when both resources are in the **same Availability Zone**. The moment you cross AZ boundaries within the same region, charges apply. This is one of the hidden AWS costs that many people overlook when designing their architecture.[3][1][2]

### Cost Impact Example

For a database-intensive application transferring **100 GB per day** between the application and database across different AZs, the monthly cost would be approximately **$60** just for data transfer (100 GB × $0.02 × 30 days). For high-traffic applications moving terabytes of data, these costs can reach hundreds of dollars monthly.[2][3]

### How to Avoid These Charges

To eliminate cross-AZ data transfer costs, place both the application server and database in the **same Availability Zone**. All data transfer within a single AZ is completely free.[1][2]

However, for production environments requiring high availability, the better approach is to deploy application servers and database replicas in **both AZs** and disable cross-zone load balancing. This way, each application server connects only to the database in its local AZ, keeping traffic free while maintaining redundancy.[4]


---
