# Index

1. [Elasticsearch Architecture](#elasticsearch-architecture)

---

## 1. Elasticsearch Architecture

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
