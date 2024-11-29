# **Role-Based Access Control (RBAC) Project**

This project demonstrates the implementation of **Role-Based Access Control (RBAC)** using **PostgreSQL** (SQL) and **MongoDB** (NoSQL). It includes database design, user roles, data population, testing, security, and backup processes.

---

## **Table of Contents**

- [Design Overview](#design-overview)
- [System Requirements](#system-requirements)
- [Setup Instructions](#setup-instructions)
  - [PostgreSQL Setup](#postgresql-setup)
  - [MongoDB Setup](#mongodb-setup)
- [Database Schemas](#database-schemas)
- [Data Population](#data-population)
- [Testing and Validation](#testing-and-validation)
- [Security](#security)
- [Backups and Restoration](#backups-and-restoration)
- [Conclusion](#conclusion)

---

## **1. Design Overview**

### **SQL (PostgreSQL):**
- **Normalized Relational Schema**: Includes tables for `organizations`, `users`, `warehouses`, and `inventory_items`.
- **Role-Based Access Control**: Uses PostgreSQL roles (`managerRole`, `staffRole`) to restrict database actions based on user responsibilities.

### **NoSQL (MongoDB):**
- **Document-Based Collections**: Stores similar data (`organizations`, `users`, `warehouses`, `inventory_items`) in collections, optimized for hierarchical access.
- **Role-Based Permissions**: MongoDB roles (`managerRole`, `staffRole`) control access to collections and actions.

---

## **2. System Requirements**

### **Software Requirements**
- **PostgreSQL**: Download from [PostgreSQL official website](https:--www.postgresql.org/download/).
- **MongoDB**: Download from [MongoDB Community Edition](https:--www.mongodb.com/docs/manual/tutorial/install-mongodb-on-windows/).
- **Visual Studio Code**: For script editing and database management.
- **Git**: For version control and repository management.

---

## **3. Setup Instructions**

### **PostgreSQL Setup**

1. **Download and Install PostgreSQL**:
   - Visit [PostgreSQL](https:--www.postgresql.org/download/) and download the installer.
   - Follow the installation wizard:
     - Set a **password for the superuser (`postgres`)** during installation.
     - Leave the default port as `5432`.

2. **Connect to PostgreSQL**:
   - Open the **SQL Shell (psql)**:
     - Host: `localhost`
     - Database: `postgres` (default)
     - Username: `postgres` (default)
     - Password: Enter the password you set during installation.

3. **Install PostgreSQL Extension in VSCode**:
   - Search for and install the **PostgreSQL** extension in VSCode.
   - Click the PostgreSQL icon in the sidebar, click the `+` icon to add a new connection, and enter:
     - Host: `localhost`
     - Username: `postgres`
     - Password: Your password
     - Port: `5432`
   - Select **Show all databases** and name your connection.

---

### **MongoDB Setup**

1. **Download and Install MongoDB**:
   - Visit [MongoDB Community Edition](https:--www.mongodb.com/docs/manual/tutorial/install-mongodb-on-windows/) and install MongoDB.
   - During installation, select **Install MongoDB Compass** (a GUI for MongoDB).

2. **Download and Set Up MongoDB Shell**:
   - Follow the instructions [here](https:--www.mongodb.com/docs/mongodb-shell/install/) to install MongoDB Shell (`mongosh`).
   - Add `mongosh` to your system's PATH.

3. **Connect MongoDB to VSCode**:
   - Install the **MongoDB for VSCode** extension.
   - Click the MongoDB icon in the sidebar and connect to `localhost`.

---

## **4. Database Schemas**

## **PostgreSQL**

### CREATE DATABASE

```sql
CREATE DATABASE warehouse
```

#### **Organizations Table**
```sql
CREATE TABLE organizations (
    org_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```
### Users Table
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    role VARCHAR(50) CHECK (role IN ('Manager', 'Staff')),
    org_id INT REFERENCES organizations(org_id) ON DELETE CASCADE
);
```
### Warehouses Table
```sql
CREATE TABLE warehouses (
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    location VARCHAR(100),
    org_id INT REFERENCES organizations(org_id) ON DELETE CASCADE
);
```
### Inventory Table
```sql
CREATE TABLE inventory_items (
    item_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    sku VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    quantity INT CHECK (quantity >= 0),
    warehouse_id INT REFERENCES warehouses(warehouse_id) ON DELETE CASCADE,
    org_id INT REFERENCES organizations(org_id) ON DELETE CASCADE
);
```

## **MongoDB**
### Collections
- organizations
- users
- warehouses
- inventory_items

### CREATE DATABASE
```nosql
use warehouses
```

## **5. Data Population**

## **PostgreSQL**
** Run the following SQL scripts:**
```sql
-- Insert Organizations
INSERT INTO organizations (name) VALUES
('Org Alpha'),
('Org Omega'),
('Org Gamma');

-- Insert Users
INSERT INTO users (name, role, org_id) VALUES
('Alice', 'Manager', 1),
('Bob', 'Staff', 1),
('Tommy', 'Manager', 2),
('Charles', 'Staff', 2),
('Henry', 'Manager', 3),
('Samantha', 'Staff', 3),
('Eve', 'Staff', 1),
('John', 'Manager', 1),
('Linda', 'Staff', 2),
('Victor', 'Manager', 3);

-- Insert Warehouses
INSERT INTO warehouses (name, location, org_id) VALUES
('Main Warehouse', 'Ilorin', 1),
('Secondary Warehouse', 'Ogun', 1),
('Main Warehouse', 'Lagos', 2),
('Secondary Warehouse', 'Oyo', 2),
('Main Warehouse', 'Abuja', 3);

-- Insert Inventory Items

DO $$
BEGIN
    FOR i IN 1..500 LOOP
        INSERT INTO inventory_items (name, sku, description, quantity, warehouse_id, org_id)
        VALUES (
            'Item ' || i,
            'SKU' || LPAD(i::TEXT, 3, '0'),
            'Description for Item ' || i,
            TRUNC(random() * 500)::INT + 10,  -- Random quantity between 10 and 500
            (i % 5) + 1,  -- Randomly assign warehouse_id (1-5)
            (i % 3) + 1   -- Randomly assign org_id (1-3)
        );
    END LOOP;
END $$;

```

## **MongoDB**
**Run the following script using `mongosh`:**
```sql
-- Connect to the database
const db = connect("mongodb:--127.0.0.1:27017/warehouse_db"); -- Adjust if using a different connection string

-- Populate Organizations
db.organizations.insertMany([
  { _id: "org1", name: "Org Alpha" },
  { _id: "org2", name: "Org Sholly" },
  { _id: "org3", name: "Org Mamood" }
]);

-- Populate Users
db.users.insertMany([
  { _id: "user1", name: "Alice", role: "Manager", organization_id: "org1" },
  { _id: "user2", name: "Bob", role: "Staff", organization_id: "org1" },
  { _id: "user3", name: "Tommy", role: "Manager", organization_id: "org2" },
  { _id: "user4", name: "Charles", role: "Staff", organization_id: "org2" },
  { _id: "user5", name: "Henry", role: "Manager", organization_id: "org3" },
  { _id: "user6", name: "Samantha", role: "Staff", organization_id: "org3" },
  { _id: "user7", name: "Eve", role: "Staff", organization_id: "org1" },
  { _id: "user8", name: "John", role: "Manager", organization_id: "org1" },
  { _id: "user9", name: "Linda", role: "Staff", organization_id: "org2" },
  { _id: "user10", name: "Victor", role: "Manager", organization_id: "org3" }
]);

-- Populate Warehouses
db.warehouses.insertMany([
  { _id: "wh1", name: "Main Warehouse", location: "Ilorin", organization_id: "org1" },
  { _id: "wh2", name: "Secondary Warehouse", location: "Ogun", organization_id: "org1" },
  { _id: "wh3", name: "Main Warehouse", location: "Lagos", organization_id: "org2" },
  { _id: "wh4", name: "Secondary Warehouse", location: "Oyo", organization_id: "org2" },
  { _id: "wh5", name: "Main Warehouse", location: "Abuja", organization_id: "org3" }
]);

-- Populate Inventory Items
for (let i = 1; i <= 500; i++) {
  const item = {
    _id: `item${i}`,
    name: `Item ${i}`,
    sku: `SKU${String(i).padStart(3, '0')}`,
    description: `Description for Item ${i}`,
    quantity: Math.floor(Math.random() * 500) + 10, -- Random quantity between 10 and 500
    warehouse_id: `wh${(i % 5) + 1}`, -- Assign warehouses cyclically (1-5)
    organization_id: `org${(i % 3) + 1}` -- Assign organizations cyclically (1-3)
  };
  db.inventory_items.insertOne(item);
}

print("Data population completed!");
```

## **6. Write Queries**
## **postgreSQL**
```sql
-- Query 1: Retrieve All Items from a Specific Organization's Warehouse
SELECT * 
FROM inventory_items
WHERE org_id = 1 AND warehouse_id = 1;

-- Query 2: Fetch All Items Below a Certain Quantity Threshold

SELECT * 
FROM inventory_items
WHERE quantity < 50;

-- Query 3: Search for Items by Name or Partial Name

SELECT * 
FROM inventory_items
WHERE name ILIKE '%Widget%';

-- Query 4: Fetch Total Number of Items per Warehouse

SELECT warehouse_id, COUNT(*) AS total_items, SUM(quantity) AS total_quantity
FROM inventory_items
GROUP BY warehouse_id;
```
## **MongoDB**
```sql

-- Retrieve All Items from a Specific Organization's Warehouse

db.inventory_items.find({ organization_id: "org1", warehouse_id: "wh1" });

-- Fetch All Items Below a Certain Quantity Threshold

db.inventory_items.find({ quantity: { $lt: 50 } });

-- Search for Items by Name or Partial Name

db.inventory_items.find({ name: { $regex: "Item 1", $options: "i" } });

-- Fetch Total Number of Items per Warehouse

db.inventory_items.aggregate([
  {
    $group: {
      _id: "$warehouse_id",
      total_items: { $sum: 1 },
      total_quantity: { $sum: "$quantity" }
    }
  }
]);

```
## **7. Add Indexes for Optimization**
## **postgreSQL**
```SQL

-- Enable the pg_trgm Extension

CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- index

CREATE INDEX idx_inventory_org_id ON inventory_items(org_id);

CREATE INDEX idx_inventory_quantity ON inventory_items(quantity);

CREATE INDEX idx_inventory_name ON inventory_items USING gin (name gin_trgm_ops);
```

## **7. Add Indexes for Optimization**
## **MongoDB**
```SQL

-- Index on organization_id

db.warehouses.createIndex({ organization_id: 1 });
db.inventory_items.createIndex({ organization_id: 1 });
db.users.createIndex({ organization_id: 1 });

-- Index on warehouse_id

db.inventory_items.createIndex({ warehouse_id: 1 });

--  Index on quantity

db.inventory_items.createIndex({ quantity: 1 });

-- Text Index on name

db.inventory_items.createIndex({ name: "text" });

-- Test impact with explain

db.inventory_items.find({ organization_id: "org1" }).explain("executionStats");

```
Indexes were chosen to optimize the performance of common queries and operations based on the expected usage patterns of the database.

# **8. Backup and Recovery Strategy**

This section outlines the proposed backup and recovery strategies for both **PostgreSQL (SQL)** and **MongoDB (NoSQL)** databases, with a focus on ensuring data integrity, availability, and consistency.

---

## **SQL Database Backup and Recovery Strategy (PostgreSQL)**

### **1. Backup Strategy**

For PostgreSQL, we will implement a **full backup** strategy along with **incremental backups** to ensure data is backed up regularly while minimizing storage usage.

#### **Full Backups**:
- **Frequency**: Full backups will be performed **weekly** to create a baseline copy of the database.
- **Tools**: We will use the `pg_dump` utility for creating a full backup of the database.
- **Backup Location**: Backups will be stored in a secure off-site or cloud location (e.g., AWS S3, Google Cloud Storage).
  
Example command for full backup:
```sql
pg_dump -U postgres -d warehouse -f C:/pgbackup/warehouse_backup.sql

```
### **1. Restore**

```sql
pg_restore -U postgres -d warehouse -f C:/pgbackup/warehouse_backup.sql
```

---

## **NoSQL Database Backup and Replication Strategy (MongoDB)**

### **1. Backup Strategy**

MongoDB requires a flexible backup strategy due to its document-oriented nature and ability to scale horizontally.

#### **Full Backups**:
- **Frequency**: Full backups will be performed **weekly**. This ensures that a snapshot of the entire database is available for recovery.
- **Tools**: We will use the `mongodump` utility to create a full backup of the MongoDB database.
- **Backup Location**: Full backups will be stored in a secure off-site or cloud storage location (e.g., AWS S3, Google Cloud Storage).
  
Example command for full backup:
```sql
mongodump --db warehouses --out C:\backup
```

### **Replication**

```sql

-- Locate your mongod.conf file and add the following under the replication section:

replication:
  replSetName: "rs0"

-- Connect to the MongoDB shell on the primary node and initialize the replica set:

rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "127.0.0.1:27017" },
    { _id: 1, host: "127.0.0.1:27018" },
    { _id: 2, host: "127.0.0.1:27019" }
  ]
});

```

## **7. Security**
## **postgreSQL**

```sql
-- Create Manager Role
CREATE ROLE manager_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO manager_role;

-- Create Staff Role
CREATE ROLE staff_role;
GRANT SELECT, UPDATE ON ALL TABLES IN SCHEMA public TO staff_role;

-- 1. Create the Users
CREATE USER alice WITH PASSWORD 'password';
CREATE USER bob WITH PASSWORD 'password';

-- 2. Grant Roles to the Users
GRANT manager_role TO alice;
GRANT staff_role TO bob;
```

## **MongoDB**

```sql

-- enable authentication:

security:
  authorization: "enabled"

-- Switch to the admin database

use admin;

-- use admin;

-- Create an admin access

db.createUser({
  user: "admin",
  pwd: "adminPassword",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
});

```

**Create Database Roles**

```sql

-- Manager Role: Full access (read, write, delete).

use warehouse_db;
db.createRole({
  role: "managerRole",
  privileges: [
    { resource: { db: "warehouse_db", collection: "" }, actions: ["find", "insert", "update", "remove"] }
  ],
  roles: []
});

-- taff Role: Limited access (read, update only)

db.createRole({
  role: "staffRole",
  privileges: [
    { resource: { db: "warehouse_db", collection: "" }, actions: ["find", "update"] }
  ],
  roles: []
});


-- Create Users and Assign Roles: Manger

db.createUser({
  user: "Alice",
  pwd: "Password",
  roles: [{ role: "managerRole", db: "warehouse_db" }]
});

-- Create Users and Assign Roles: Staff

db.createUser({
  user: "Bob",
  pwd: "Password",
  roles: [{ role: "staffRole", db: "warehouse_db" }]
});

```
---

## **Conclusion**

This project demonstrates the effective implementation of **Role-Based Access Control (RBAC)** for both **SQL (PostgreSQL)** and **NoSQL (MongoDB)** databases. By implementing roles such as `Manager` and `Staff`, we can ensure that users have appropriate access to data based on their responsibilities.

### Key Takeaways:
- **PostgreSQL** utilizes a **normalized relational schema** and leverages roles to control access, ensuring that users can only perform permitted actions on the database.
- **MongoDB** implements **document-based collections** with role-based permissions, enabling scalable, hierarchical access control.
- **Data Population**: Data is populated using SQL for PostgreSQL and MongoDB scripts for NoSQL, which supports both relational and non-relational data models.
- **Security**: Role-based security ensures that sensitive data is protected, and users can only access or modify what they are authorized to.
- **Backup and Recovery**: A robust backup strategy is in place, with **full and incremental backups** for PostgreSQL and **oplog-based backups** for MongoDB. Replication ensures high availability and fault tolerance for MongoDB.

By following these practices, we ensure **data integrity**, **availability**, and **security** across both databases, making the system resilient, efficient, and easy to manage.

---
