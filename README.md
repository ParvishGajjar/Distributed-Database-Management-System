# Distributed Database Management System (DDBMS)

## Overview
This repository contains the implementation of a Distributed Database Management System (DDBMS) built in Java. The primary goal of the project is to simulate the functionality of a distributed relational database system with features similar to SQL Server Management Studio (SSMS). The system allows users to create databases and tables, insert and query data, and retrieve information distributed across multiple virtual machines (VMs). 

### Project Highlights
- Implements a custom SQL-like query language for database interaction.
- Designed to operate in a distributed environment using multiple VMs.
- Stores data using scalable file-based storage (tilde-separated `~` TSV files).
- Enables inter-node communication to fetch and process data.
- Supports metadata management via a Global Data Dictionary (GDD).

## Key Features
### 1. **Distributed Architecture**
- Tables within a database can be distributed across multiple VMs.
- Query execution handles data locality transparently—queries on remote tables fetch data across nodes using HTTP communication.
- Metadata maintained in the GDD (`gdd.tsv`) provides node-to-table and database mapping.

### 2. **Custom Query Language**
The system provides an easy-to-use SQL-inspired language that supports:
- `CREATE DATABASE <name>`: Creates a new database.
- `USE <database>`: Sets the active database context.
- `CREATE TABLE <name> (<columns>)`: Defines a new table with specified schema.
- `INSERT INTO <table> VALUES (<values>)`: Adds rows of data to the specified table.
- `SELECT <columns> FROM <table>`: Fetches records from a table.
- `UPDATE` and `DELETE` operations for data manipulation.

### 3. **File-Based Storage**
- Tables are stored in `.tsv` files where columns are separated using a tilde (`~`) character.
- Metadata for table properties is defined in `metadata.tsv` files.

### 4. **High-Level Components**
- **Metadata Management**: Tracks and assigns tables/databases to specific nodes.
- **Query Distribution**: Routes queries to the appropriate nodes based on metadata.
- **Networking**: REST API endpoints provide inter-node communication for table reads, updates, and metadata synchronization.

## Repository Structure
Below is the top-level structure of the repository:

```
/src
  /main
    /java
      /com/example/project
        /DistributedDatabaseLayer
          - Requester.java    # Handles inter-node REST API calls
          - Responder.java    # Responds to external HTTP queries
        /QueryManager
          - DatabaseProcessor.java  # Manages database-related operations
          - TableProcessor.java     # Processes table-level queries
        /Utils
          - Utils.java        # Common utilities (delimiters, logging, paths)
  /test
    ...                       # Test files (if implemented)

README.md                    # General project documentation
pom.xml                      # Maven build configuration
gdd.tsv                      # Sample global data dictionary file for metadata
```

## Implementation Details
### Query Processing
1. **Parsing and Execution**:
   - Queries are parsed to identify their structure (e.g., SELECT vs CREATE).
   - If the request is for a remote table, the query is forwarded to the appropriate VM.

2. **Requester Class**:
   - Sends HTTP requests to remote nodes to fetch or manipulate data.
   - Supports operations like `requestVMDBCheck`, `requestVMInsertQuery`, `requestVMSelectQuery`.

3. **Responder Class**:
   - Hosts REST endpoints (e.g., `/query`) to process distributed queries and communicate responses.

### Metadata Management
- Metadata for databases and tables is stored in `gdd.tsv`.
- `DatabaseProcessor` and `TableProcessor` manage metadata lookup and ensure correct database/table retrieval.

### Distributed Query Execution
1. **Local Queries**:
   - Processed directly on the VM where the database/table resides.

2. **Remote Queries**:
   - Metadata determines the table's host node.
   - Query is forwarded to the relevant VM using the `Requester`.

3. **Global Operations**:
   - Certain queries (e.g., `CREATE DATABASE`) update metadata across nodes to maintain consistency.

## Tilde-Separated Values (TSV) File Format
Tables are stored in `.tsv` files with the following properties:
- Rows are separated by newlines (`\n`).
- Columns in a row are separated by the tilde (`~`) character.

Example table (`users.tsv`):
```
id~name~email
1~Alice~alice@example.com
2~Bob~bob@example.com
```

## Build and Run Instructions
### Prerequisites
- Java Development Kit (JDK) 8 or higher.
- Maven build tool (or Maven wrapper).

### Build Commands
To compile the project:
```bash
./mvnw clean install
```

### Running a Node
To start a node, run:
```bash
java -cp target/classes com.example.project.Main --id VM1 --port 8080 --metadata /path/to/gdd.tsv
```
- **`--id`**: Unique identifier for the node.
- **`--port`**: Port for REST API communication.
- **`--metadata`**: Path to the `gdd.tsv` file.

### Query Execution
For simplicity, use the CLI interface to execute commands:
```
CREATE DATABASE testDB;
USE testDB;
CREATE TABLE users (id INT, name STRING);
INSERT INTO users VALUES (1, 'Alice');
SELECT * FROM users;
```

## Example Workflow
1. VM1 processes `CREATE DATABASE` and updates `gdd.tsv` for global consistency.
2. Distributed queries are forwarded to the appropriate VM based on metadata.
3. Results from remote tables are fetched via HTTP:
   - VM1 queries `VM2` for `users` table rows.
   - VM2 responds with serialized table data.

## REST Endpoints
Exposed endpoints (handled by `Responder.java`):
- `/query`: Accepts SELECT, INSERT, UPDATE, DELETE.
- `/gdd`: Handles metadata queries like `checkDB`.

## Future Enhancements
- **High Availability**:
  - Introduce replication for fault tolerance.
- **Distributed Transactions**:
  - Implement two-phase commit for atomicity.
- **Improved Query Planner**:
  - Add logic for query optimization across nodes.
- **Security**:
  - Incorporate authentication and encrypted communication.

## Contributing
If you’d like to contribute:
1. Fork the repository.
2. Create a feature branch.
3. Submit a PR with detailed explanations.

For assistance or bug reports, contact the repository owner.

## License
This project does not specify a license. Please add an appropriate open-source license (e.g., MIT, Apache 2.0) for external contributions.