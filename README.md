# 📱 Phone Book Management System

A robust, multi-user contact management application developed in **C++** with a **MySQL** database backend. The system enables users to efficiently manage, group, prioritize, and search contact records securely while providing administrative capabilities for account management and system recovery.

---

## 📑 Table of Contents
- [Key Features](#-key-features)
  - [User Features](#user-features)
  - [Admin Features](#admin-features)
- [System Architecture](#-system-architecture)
  - [Data Model & ERD](#data-model--erd)
  - [Class Hierarchy](#class-hierarchy)
- [Tech Stack & System Requirements](#-tech-stack--system-requirements)
- [Installation & Getting Started](#-installation--getting-started)
  - [Prerequisites](#prerequisites)
  - [Database Setup](#database-setup)
  - [Build and Run](#build-and-run)
- [Security & Performance Guidelines](#-security--performance-guidelines)
- [Project Changelog](#-project-changelog)
- [License](#-license)

---

## ✨ Key Features

### 👤 User Features
- **Account Management:**
  - Secure registration, authentication, profile updates, and password changes.
  - Session termination upon logout.
- **Contact Management (CRUD):**
  - **Add Contact:** Store full name, phone number, email, address, and group associations with input validation.
  - **View & Pagination:** Browse contacts with summary views; handles empty data states gracefully.
  - **Edit & Delete:** Update existing contacts or delete with mandatory confirmation prompts.
- **Advanced Search & Filtering:**
  - Search contacts by keyword (Full Name, Phone Number, or Email).
  - Alphabetical (A–Z) sorting and group-based filtering.
- **Group Categorization:**
  - Create, view, edit, and delete custom contact groups (e.g., *Family, Coworkers, Clients*).
  - Assign contacts to one or multiple groups (many-to-many relationship).
- **Favorites / Priority Flagging:**
  - Flag frequently interacted or VIP contacts with a star/favorite mark for quick access.

### 🛡️ Admin Features
- **User Administration:** View user directory, update user profiles, create new users, or revoke/delete accounts.
- **System Reports:** Monitor overall platform metrics including total users, active contacts, and groups.
- **Backup & Recovery:**
  - On-demand system snapshots (`accounts`, `contacts`, `groups`, `contact_group`).
  - File restoration with confirmation dialogs to prevent data loss.

---

## 🏗️ System Architecture

### 📊 Data Model & ERD

The relational database is built on 4 core tables to ensure complete data isolation across users:

```
+--------------------+            1:N            +--------------------+
|      Accounts      |-------------------------->|      Contacts      |
|--------------------|                           |--------------------|
| PK account_id      |                           | PK contact_id      |
|    username        |                           |    name            |
|    password (hash) |                           |    phone           |
|    fullname        |                           |    email           |
|    email           |                           |    address         |
|    phone           |                           |    is_favorite     |
|    role            |                           | FK account_id      |
+--------------------+                           +--------------------+
          | 1                                               | 1
          |                                                 |
          | 1:N                                             | M:N
          v                                                 v
+--------------------+                           +----------------------+
|       Groups       |<--------------------------|    Contact_Group     |
|--------------------|            1:N            |----------------------|
| PK group_id        |                           | PK, FK contact_id    |
|    group_name      |                           | PK, FK group_id      |
|    description     |                           +----------------------+
| FK account_id      |
+--------------------+
```

- **Accounts $\rightarrow$ Contacts & Groups:** $1:N$ relationship with `ON DELETE CASCADE`. Users can only access entities where `account_id` matches their authenticated session.
- **Contacts $\leftrightarrow$ Groups:** $N:N$ relationship decoupled via the intermediate `Contact_Group` table.

---

### 🧩 Class Hierarchy

```
       +-----------------------+
       |        Account        |
       |-----------------------|
       | - accountId: int      |
       | - username: string    |
       | - password: string    |
       | - role: string        |
       +-----------------------+
              ^         ^
              |         | (Inheritance)
     +--------+         +--------+
     |                           |
+---------+                 +---------+
|  User   |                 |  Admin  |
+---------+                 +---------+
  ◆     ◆
  |     | (Composition)
  |     +---------------> [Group] (groupId, groupName, description)
  v
[Contact] (contactId, name, phone, email, address, isFavorite)
```

---

## 🛠️ Tech Stack & System Requirements

- **Language:** C++ (C++11 or higher recommended)
- **Database:** MySQL 5.7+ / 8.0+
- **Driver:** MySQL Connector/C++ (`libmysqlclient-dev` / `mysql-connector-c++`)
- **Target OS:** CentOS 6.0+ / Linux / POSIX-compliant environments

---

## 🚀 Installation & Getting Started

### Prerequisites

On CentOS/RHEL systems:
```bash
sudo yum update -y
sudo yum install -y gcc-c++ make mysql-server mysql-devel
```

On Debian/Ubuntu systems:
```bash
sudo apt update
sudo apt install -y build-essential libmysqlclient-dev default-mysql-server
```

### Database Setup

1. Start the MySQL service:
   ```bash
   sudo systemctl start mysql   # or `service mysqld start` on CentOS 6
   ```

2. Initialize the database schema:
   ```sql
   CREATE DATABASE IF NOT EXISTS phonebook_db;
   USE phonebook_db;

   CREATE TABLE Accounts (
       account_id INT AUTO_INCREMENT PRIMARY KEY,
       username VARCHAR(50) NOT NULL UNIQUE,
       password VARCHAR(255) NOT NULL,
       fullname VARCHAR(100) NOT NULL,
       email VARCHAR(100) UNIQUE,
       phone VARCHAR(15),
       role VARCHAR(20) NOT NULL DEFAULT 'user'
   );

   CREATE TABLE Contacts (
       contact_id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       phone VARCHAR(15) NOT NULL,
       email VARCHAR(100),
       address VARCHAR(255),
       is_favorite BOOLEAN NOT NULL DEFAULT FALSE,
       account_id INT NOT NULL,
       FOREIGN KEY (account_id) REFERENCES Accounts(account_id) ON DELETE CASCADE
   );

   CREATE TABLE Groups (
       group_id INT AUTO_INCREMENT PRIMARY KEY,
       group_name VARCHAR(100) NOT NULL,
       description VARCHAR(255),
       account_id INT NOT NULL,
       FOREIGN KEY (account_id) REFERENCES Accounts(account_id) ON DELETE CASCADE
   );

   CREATE TABLE Contact_Group (
       contact_id INT NOT NULL,
       group_id INT NOT NULL,
       PRIMARY KEY (contact_id, group_id),
       FOREIGN KEY (contact_id) REFERENCES Contacts(contact_id) ON DELETE CASCADE,
       FOREIGN KEY (group_id) REFERENCES Groups(group_id) ON DELETE CASCADE
   );
   ```

### Build and Run

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/phone-book-management-system.git
   cd phone-book-management-system
   ```

2. Compile the source code:
   ```bash
   g++ -std=c++11 -Wall src/*.cpp -o phonebook_app $(mysql_config --cflags --libs)
   ```

3. Configure your database credentials in the configuration file or environment variables, then execute:
   ```bash
   ./phonebook_app
   ```

---

## 🔒 Security & Performance Guidelines

- **Password Hashing:** Passwords must be hashed (e.g., SHA-256 or bcrypt) prior to storage; plain text is strictly forbidden.
- **Data Isolation:** All read/write queries enforce `WHERE account_id = ?` filters to ensure multi-tenant boundary integrity.
- **SQL Injection Defense:** Utilize parameterized queries or prepared statements across all input boundaries.
- **High Concurrency Target:** Engineered to support $10,000+$ concurrent users and over $1,000,000+$ contact records with $\ge 99.9\%$ system availability.

---

## 📝 Project Changelog

| Date | Version | Summary of Changes |
| :--- | :--- | :--- |
| **10/08/2025** | `1.0` | Initial Documentation Development |
| **15/08/2025** | `1.0` | Specification of Functional and Non-Functional Requirements |
| **19/08/2025** | `1.0` | Initial DFD, Use Case, Class Diagram, Data Model & UI Design |
| **21/08/2025** | `1.1` | Refined Diagrams and Interface Specifications |
| **22/08/2025** | `1.2` | Finalized Class Diagram, Data Model, and Mockup Specifications |

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
