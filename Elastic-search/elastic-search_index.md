# Index

1. [Elasticsearch Architecture](#elasticsearch-architecture)
2. [Sharding](#sharding-in-elasticsearch)

---

## Elasticsearch Architecture

- Youtube link to intro: https://www.youtube.com/watch?v=Bdt8M_RwHVs&list=PLA3GkZPtsafYd5m2BXmkL9pjsBKy0FQ2X&index=3
- Documentation: 
---
### Anology to Learn elasticsearch as database:

- **Document** → **Row in SQL**: Each document in Elasticsearch corresponds to a single record or row in a SQL database table.
- **Fields** → **Columns in SQL**: Fields in a document are similar to columns in a SQL table; they represent attributes or data points for a specific document.
- **Index** → **Table in SQL**: An index in Elasticsearch is analogous to a table in SQL; it contains multiple documents (rows) with similar structures.
---
#### **Query to Add Data**
- **`POST`**: Creates or updates a document.
- **`/employees/_doc/1`**: Indicates the `employees` index, `_doc` type (default), and the document ID `1`.

```bash
POST /employees/_doc/1
{
  "id": "1",
  "name": "John Doe",
  "position": "Software Engineer",
  "department": "Engineering",
  "hire_date": "2021-06-15",
  "skills": ["Java", "Python", "Elasticsearch"],
  "salary": 90000
}
```
---

### **Retrieving Data and Metadata**
When you retrieve the data, Elasticsearch includes metadata in the response. Here's an example query and the expected response:

#### **Query to Retrieve Data**
```bash
GET /employees/_doc/1
```

---

### **Response**
```json
{
  "_index": "employees",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "id": "1",
    "name": "John Doe",
    "position": "Software Engineer",
    "department": "Engineering",
    "hire_date": "2021-06-15",
    "skills": ["Java", "Python", "Elasticsearch"],
    "salary": 90000
  }
}
```

---

### **Metadata in the Response**
1. **`_index`**:
   - The name of the index where the document is stored (e.g., `employees`).

2. **`_id`**:
   - The unique identifier of the document (e.g., `1`).

3. **`_version`**:
   - Tracks the version of the document. Incremented with each update.

4. **`_seq_no`**:
   - Sequence number for the operation. Used for conflict resolution.

5. **`_primary_term`**:
   - Primary term of the shard. Used with `_seq_no` to ensure consistency.

6. **`found`**:
   - Indicates whether the document was found (`true` or `false`).

7. **`_source`**:
   - Contains the actual document data that was indexed.

---

### **Extra Metadata Explanation**
- **Operational Metadata** (`_seq_no`, `_primary_term`): Helps maintain consistency in distributed systems.
- **Versioning** (`_version`): Enables optimistic concurrency control, allowing safe updates.
- **Index Information** (`_index`, `_id`): Identifies where the document resides.

---

### **Key Points**
- Metadata like `_seq_no` and `_primary_term` are useful for conflict resolution in distributed environments.
- `_version` helps you track document changes, especially during updates.
- `_source` contains the user-supplied JSON data, which is the most commonly accessed part of the response. 

This combination of data and metadata makes Elasticsearch highly efficient for managing, searching, and maintaining document consistency in distributed systems.

---

## Sharding in Elasticsearch

**Sharding** is a fundamental concept in Elasticsearch that allows you to **divide an index into smaller, more manageable pieces** called **shards**. These shards distribute the data and load across multiple nodes in the cluster, enabling Elasticsearch to scale horizontally and handle large amounts of data.

---

### **How Sharding Works**
1. **Dividing the Data**:
   - When you create an index in Elasticsearch, it is divided into multiple shards.
   - Each shard contains a subset of the data in the index and is a self-contained, fully functional index.

2. **Distributing the Shards**:
   - Shards are distributed across the nodes in a cluster to:
     - Balance the load (query and indexing).
     - Provide redundancy for fault tolerance (via replicas).

3. **Query Execution**:
   - When you query an index, Elasticsearch sends the query to all the shards for that index.
   - Each shard processes its portion of the data, and the results are combined and returned to the user.

---

### **Level of Sharding**
Sharding is done at the **index level**:
- An **index** is divided into multiple **primary shards**.
- For each primary shard, Elasticsearch can create **replica shards**.

---

### **Terminologies Related to Sharding**

| **Term**          | **Definition**                                                                                      |
|--------------------|----------------------------------------------------------------------------------------------------|
| **Shard**          | A smaller unit of an index that stores part of the index’s data and acts as a fully functional index.|
| **Primary Shard**  | The original shard responsible for storing data. Each document is stored in one and only one primary shard. |
| **Replica Shard**  | A copy of a primary shard, used for redundancy and high availability. Replica shards also handle read requests. |
| **Node**           | A single instance of Elasticsearch running in a cluster. Shards are distributed across nodes.      |
| **Cluster**        | A collection of nodes that work together to store and search data.                                 |
| **Index**          | A collection of documents that share similar characteristics (e.g., a database table in SQL).      |
| **Routing**        | The process Elasticsearch uses to determine which shard a document or query is assigned to.         |

---

### **Sharding and Scaling**
1. **Horizontal Scaling**:
   - By increasing the number of shards, you can distribute data across more nodes.
   - This enables Elasticsearch to handle larger datasets and higher query throughput.

2. **Parallel Processing**:
   - Queries and indexing operations are parallelized across shards, improving performance.

---

### **Default Sharding Configuration**
- By default, an index in Elasticsearch has **5 primary shards** and **1 replica per shard**. This configuration can be adjusted based on:
  - The size of the data.
  - Query volume.
  - Cluster resources.

---

### **Important Considerations**
1. **Shard Size**:
   - Each shard should ideally be between **10GB and 50GB** for optimal performance.
   - Too many small shards can lead to overhead, while very large shards can slow down queries.

2. **Replication**:
   - Replicas improve **fault tolerance** and **read scalability** (queries can be served by replicas).

3. **Re-sharding**:
   - Changing the number of shards after the index is created is complex and typically requires **index reindexing**.

---

### **Practical Example**
Suppose you create an index `user_data` with:
- **3 primary shards**
- **1 replica per shard**

If you have a 3-node Elasticsearch cluster:
1. Each node will hold **1 primary shard** and **1 replica shard**.
2. Data is evenly distributed, and queries can be handled by both primary and replica shards.

---

### **Summary**
- Sharding divides an **index** into smaller **primary shards**.
- Each shard is stored on a **node**, allowing distributed storage and processing.
- Replicas are used for redundancy and query performance.
- Proper sharding configuration is crucial for scalability and performance in Elasticsearch.

In Elasticsearch, **Split API** and **Shrink API** are used to adjust the number of primary shards in an index. These APIs help you manage the shard configuration for scaling purposes.

---

### **1. Split API**
The **Split API** is used to increase the number of primary shards for an index. It is helpful when:
- The data in an index grows significantly, causing the existing shards to become too large.
- You want to scale out an index for better query performance.

#### **How It Works**
The **Split API** divides each primary shard into **multiple new shards**. For example, an index with 1 primary shard can be split into 2, 4, or more shards (a power of 2).

#### **Key Requirements**
1. The index must be in a **read-only** state before splitting.
2. The new shard count must be a multiple of the current shard count (e.g., 1 → 2, 2 → 4, 4 → 8).
3. You need to set the `index.number_of_shards` setting to the desired shard count.

#### **Steps to Use Split API**
1. **Set Index to Read-Only**:
   ```bash
   PUT /my_index/_settings
   {
     "settings": {
       "index.blocks.write": true
     }
   }
   ```

2. **Split the Index**:
   ```bash
   POST /my_index/_split/my_new_index
   {
     "settings": {
       "index.number_of_shards": 4
     }
   }
   ```

3. **Verify the New Index**:
   ```bash
   GET /my_new_index/_settings
   ```

4. **Re-enable Writing (Optional)**:
   ```bash
   PUT /my_new_index/_settings
   {
     "settings": {
       "index.blocks.write": false
     }
   }
   ```

---

### **2. Shrink API**
The **Shrink API** is used to reduce the number of primary shards for an index. It is useful when:
- The index’s data volume is small and the shards are underutilized.
- You want to consolidate shards for better resource efficiency.

#### **How It Works**
The **Shrink API** merges multiple primary shards into fewer, larger shards. For example, an index with 8 shards can be shrunk to 4, 2, or 1 shard.

#### **Key Requirements**
1. The index must be in a **read-only** state before shrinking.
2. The total number of primary shards must be divisible by the target shard count (e.g., 8 → 4, 4 → 2).
3. All primary shards must be on the same node.

#### **Steps to Use Shrink API**
1. **Set Index to Read-Only**:
   ```bash
   PUT /my_index/_settings
   {
     "settings": {
       "index.blocks.write": true
     }
   }
   ```

2. **Allocate Shards to One Node**:
   ```bash
   PUT /my_index/_settings
   {
     "settings": {
       "index.routing.allocation.require._name": "node_name"
     }
   }
   ```

3. **Shrink the Index**:
   ```bash
   POST /my_index/_shrink/my_shrunk_index
   {
     "settings": {
       "index.number_of_shards": 2
     }
   }
   ```

4. **Verify the New Index**:
   ```bash
   GET /my_shrunk_index/_settings
   ```

5. **Re-enable Writing (Optional)**:
   ```bash
   PUT /my_shrunk_index/_settings
   {
     "settings": {
       "index.blocks.write": false
     }
   }
   ```

---

### **How They Relate to Sharding**
1. **Split API**:
   - Increases the number of primary shards.
   - Used when your data grows beyond the capacity of existing shards.
   - Helps scale out for high query throughput.

2. **Shrink API**:
   - Decreases the number of primary shards.
   - Consolidates small shards to reduce overhead and improve resource efficiency.
   - Helps optimize for small, stable datasets.

---

### **Practical Use Cases**
| **API**    | **Use Case**                                                                                         |
|------------|-----------------------------------------------------------------------------------------------------|
| **Split**  | An index has grown too large (e.g., each shard is >50GB), and you need more shards for better scaling. |
| **Shrink** | An index has too many small shards (e.g., each shard is <1GB), and you want to reduce overhead.        |

---

### **Important Notes**
1. Both APIs require the index to be **read-only** during the operation.
2. After the operation, the new index will have a new name; the old index remains unchanged unless deleted manually.
3. Carefully plan the shard configuration to avoid frequent re-sharding, as both operations can be time-consuming.

By properly using the Split and Shrink APIs, you can dynamically manage shard allocation in Elasticsearch to adapt to changing data and performance needs.
