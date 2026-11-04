# Smart Inventory Management System (Inventory Pro)

A desktop inventory management application built with **Java Swing**, **JDBC** and **MySQL**. It replaces manual, spreadsheet-based stock tracking with a role-based system where an admin manages shop owners and each shop owner manages their own suppliers, products, orders and sales.

> DBMS course project, SRM Institute of Science and Technology (Feb 2026 – Apr 2026)

## Features

- **Role-based login**: separate Admin and Shop Owner dashboards.
- **Registration with approval**: shop owners submit a registration request; the admin approves or rejects it.
- **Inventory modules**: add, update, delete and view **suppliers, products, orders and sales**.
- **Shop-wise data**: shop owners work with their own shop's inventory.
- **User management**: admin can activate or deactivate accounts.
- **Activity log**: logins and key actions are recorded and visible to the admin.
- **In-app messaging**: shop owners send queries to the admin, who can reply, resolve, or broadcast a message.
- **Polished UI**: splash screen, animated panels, gradient and rounded components.

## Tech stack

| Layer | Technology |
|---|---|
| Interface | Java Swing |
| Data access | JDBC (MySQL Connector/J) |
| Database | MySQL 8 |

## Project structure

```
Main.java                  Entry point (splash screen, then login)
DatabaseUtil.java          MySQL connection settings
LoginPanel.java            Role-based login
Session.java               Logged-in user state
Admin*/Shop* panels        Admin and shop-owner dashboards
Product / Supplier / Order / Sale panels   Inventory modules
ApprovalPanel, RegisterRequestPanel        Account approval flow
ActivityLogPanel, Message*/Broadcast panels   Audit trail and messaging
schema.sql                 Database schema (see below)
```

## Prerequisites

- JDK 11 or later
- MySQL Server 8.x
- [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/) (the `.jar` file)
- Windows is recommended, since the UI uses the *Segoe UI Emoji* font

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/shubhsachan777747/Smart_Inventory_Management_System_DBMS.git
cd Smart_Inventory_Management_System_DBMS
```

### 2. Create the database

Run the schema script in MySQL:

```bash
mysql -u root -p < schema.sql
```

This creates the `inventory_dbs` database, its tables and a first admin account (see **Default login** below).

### 3. Configure the connection

Open `DatabaseUtil.java` and set your MySQL details:

```java
private static final String URL = "jdbc:mysql://localhost:3306/inventory_dbs";
private static final String USER = "root";
private static final String PASSWORD = "your_mysql_password";
```

### 4. Compile and run

Place the Connector/J jar (for example `mysql-connector-j-8.x.x.jar`) in the project folder.

**Windows**

```bash
javac -cp ".;mysql-connector-j-8.x.x.jar" *.java
java  -cp ".;mysql-connector-j-8.x.x.jar" Main
```

**macOS / Linux**

```bash
javac -cp ".:mysql-connector-j-8.x.x.jar" *.java
java  -cp ".:mysql-connector-j-8.x.x.jar" Main
```

You can also open the folder in IntelliJ IDEA, add the Connector/J jar as a library and run `Main`.

## Default login

`schema.sql` creates one admin account:

| Role | Username | Password |
|---|---|---|
| Admin | `admin` | `admin123` |

Change this password straight away. Shop owners register from the login screen and are approved by the admin.

## Database design

| Table | Purpose |
|---|---|
| `Users` | Admin and shop-owner accounts |
| `PendingRegistrations` | Registration requests awaiting approval |
| `Supplier` | Suppliers for each shop |
| `Product` | Products with price, quantity and supplier |
| `Orders` | Supplier orders |
| `Sale` | Recorded sales |
| `Messages` | Shop-owner queries, admin replies and broadcasts |
| `ActivityLog` | Login and action history |

## Known limitations and future work

- Passwords are stored as plain text. Hash them (for example with bcrypt) before real use.
- Database credentials are set in source code. Move them to environment variables or a config file.
- Add input validation and stock alerts for low quantities.

- -- Smart Inventory Management System: database schema
-- Reconstructed from the SQL used in the Java code. Adjust column types if your database differs.

CREATE DATABASE IF NOT EXISTS inventory_dbs;
USE inventory_dbs;

CREATE TABLE IF NOT EXISTS Users (
    user_id       INT AUTO_INCREMENT PRIMARY KEY,
    username      VARCHAR(50)  NOT NULL UNIQUE,
    password      VARCHAR(100) NOT NULL,
    full_name     VARCHAR(100),
    email         VARCHAR(100) UNIQUE,
    role          ENUM('admin','shop_owner') NOT NULL DEFAULT 'shop_owner',
    shop_name     VARCHAR(100),
    shop_address  VARCHAR(255),
    phone         VARCHAR(20),
    shop_id       INT NULL,
    is_active     BOOLEAN NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS PendingRegistrations (
    request_id    INT AUTO_INCREMENT PRIMARY KEY,
    username      VARCHAR(50)  NOT NULL,
    password      VARCHAR(100) NOT NULL,
    full_name     VARCHAR(100),
    email         VARCHAR(100),
    shop_name     VARCHAR(100),
    shop_address  VARCHAR(255),
    phone         VARCHAR(20),
    status        ENUM('pending','approved','rejected') NOT NULL DEFAULT 'pending',
    processed_by  INT NULL,
    processed_at  DATETIME NULL,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS Supplier (
    Supplier_ID   INT AUTO_INCREMENT PRIMARY KEY,
    First_name    VARCHAR(50),
    Last_name     VARCHAR(50),
    Contact_info  VARCHAR(100),
    shop_id       INT NULL,
    added_by      INT NULL
);

CREATE TABLE IF NOT EXISTS Product (
    Product_ID    INT AUTO_INCREMENT PRIMARY KEY,
    Product_name  VARCHAR(100) NOT NULL,
    Price         DECIMAL(10,2) NOT NULL CHECK (Price > 0),
    Supplier_ID   INT NULL,
    Quantity      INT NOT NULL DEFAULT 0,
    added_by      INT NULL,
    FOREIGN KEY (Supplier_ID) REFERENCES Supplier(Supplier_ID)
);

CREATE TABLE IF NOT EXISTS Orders (
    Order_ID      INT AUTO_INCREMENT PRIMARY KEY,
    Supplier_ID   INT NULL,
    Order_Date    DATE,
    status        VARCHAR(30) DEFAULT 'pending',
    added_by      INT NULL,
    FOREIGN KEY (Supplier_ID) REFERENCES Supplier(Supplier_ID)
);

CREATE TABLE IF NOT EXISTS Sale (
    Sale_ID       INT AUTO_INCREMENT PRIMARY KEY,
    Product_ID    INT NULL,
    Total_sale    DECIMAL(10,2),
    sale_date     DATE,
    added_by      INT NULL,
    FOREIGN KEY (Product_ID) REFERENCES Product(Product_ID)
);

CREATE TABLE IF NOT EXISTS Messages (
    message_id    INT AUTO_INCREMENT PRIMARY KEY,
    sender_id     INT NULL,
    receiver_id   INT NULL,
    subject       VARCHAR(150),
    message_body  TEXT,
    status        VARCHAR(20) DEFAULT 'unread',
    priority      VARCHAR(20) DEFAULT 'normal',
    admin_reply   TEXT NULL,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved_at   DATETIME NULL
);

CREATE TABLE IF NOT EXISTS ActivityLog (
    log_id        INT AUTO_INCREMENT PRIMARY KEY,
    user_id       INT NOT NULL,
    activity_type VARCHAR(50),
    description   VARCHAR(255),
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);

-- First admin account (user_id 1). The app sends admin messages to user_id 1.
INSERT INTO Users (user_id, username, password, full_name, email, role)
VALUES (1, 'admin', 'admin123', 'System Admin', 'admin@example.com', 'admin')
ON DUPLICATE KEY UPDATE username = username;

## Author

**Shubh Sachan**: [GitHub](https://github.com/shubhsachan777747)
