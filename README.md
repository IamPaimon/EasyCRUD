# MariaDB Setup & Configuration Guide

A practical guide for installing **MariaDB**, securing the database server, creating the application database and user, and preparing database credentials for connecting a backend application.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [1. Install MariaDB](#1-install-mariadb)
* [2. Secure MariaDB](#2-secure-mariadb)
* [3. Create the Application Database](#3-create-the-application-database)
* [4. Create the Database User](#4-create-the-database-user)
* [5. Database Credentials](#5-database-credentials)
* [6. Create the Students Table](#6-create-the-students-table)
* [7. Verify the Database](#7-verify-the-database)
* [8. Exit MariaDB](#8-exit-mariadb)
* [Security Recommendations](#security-recommendations)

---

## Prerequisites

Before starting, make sure you have:

* A Linux server or Ubuntu system
* `sudo` or root privileges
* Internet connectivity
* MariaDB server access
* A backend application that will connect to MariaDB

> **Note:** Despite the original title referring to Windows, the installation commands in this guide are for **Ubuntu/Linux**.

---

# 1. Install MariaDB

Update the package index and install the MariaDB server:

```bash
sudo apt update
sudo apt install mariadb-server -y
```

Verify that MariaDB is installed:

```bash
mariadb --version
```

Check the MariaDB service:

```bash
sudo systemctl status mariadb
```

If the service is not running, start it:

```bash
sudo systemctl start mariadb
```

Enable MariaDB to start automatically after a system reboot:

```bash
sudo systemctl enable mariadb
```

---

# 2. Secure MariaDB

Run the MariaDB security configuration utility:

```bash
sudo mysql_secure_installation
```

Follow the prompts to secure the database server.

Recommended actions include:

* Set a root password if prompted
* Remove anonymous users
* Remove the default test database
* Remove unnecessary privileges
* Disable remote root login
* Reload privilege tables

### Why this matters

A default database installation may contain accounts or configuration that are unsuitable for a production environment.

The security script helps reduce the attack surface before the database is used by your application.

---

# 3. Create the Application Database

Log in to MariaDB:

```bash
sudo mariadb
```

Or, if your root account uses password authentication:

```bash
mysql -u root -p
```

Create the application database:

```sql
CREATE DATABASE student_db;
```

Verify that the database was created:

```sql
SHOW DATABASES;
```

You should see:

```text
student_db
```

---

# 4. Create the Database User

Create a dedicated database user for the application.

> **Important:** Do not use the MariaDB `root` account from your application.

Example:

```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'your_password';
```

Grant the required privileges on the application database:

```sql
GRANT ALL PRIVILEGES ON student_db.* TO 'username'@'localhost';
```

Apply the privilege changes:

```sql
FLUSH PRIVILEGES;
```

Verify the user's privileges:

```sql
SHOW GRANTS FOR 'username'@'localhost';
```

### Replace the following values

| Placeholder     | Replace with                       |
| --------------- | ---------------------------------- |
| `username`      | Your application database username |
| `your_password` | A strong database password         |
| `student_db`    | Your application database name     |

---

# 5. Database Credentials

Your backend application will require the following database configuration values:

| Variable  | Description                   | Example      |
| --------- | ----------------------------- | ------------ |
| `DB_HOST` | Database server hostname/IP   | `localhost`  |
| `DB_USER` | Application database user     | `username`   |
| `DB_PASS` | Application database password | `********`   |
| `DB_PORT` | MariaDB port                  | `3306`       |
| `DB_NAME` | Database name                 | `student_db` |

### Example environment configuration

```bash
DB_HOST=localhost
DB_USER=username
DB_PASS=your_password
DB_PORT=3306
DB_NAME=student_db
```

> **Security:** Never commit real database passwords, API keys, or other secrets to Git.

For production deployments, configure these values using your hosting platform's environment variables or secret-management system.

---

# 6. Create the Students Table

Select the application database:

```sql
USE student_db;
```

Create the `students` table:

```sql
CREATE TABLE `students` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `course` varchar(255) DEFAULT NULL,
  `student_class` varchar(255) DEFAULT NULL,
  `percentage` double DEFAULT NULL,
  `branch` varchar(255) DEFAULT NULL,
  `mobile_number` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB
  DEFAULT CHARSET=latin1
  COLLATE=latin1_swedish_ci;
```

Verify the table:

```sql
SHOW TABLES;
```

Check its structure:

```sql
DESCRIBE students;
```

---

# 7. Verify the Database

Select the database:

```sql
USE student_db;
```

Check the available tables:

```sql
SHOW TABLES;
```

Check the table structure:

```sql
DESCRIBE students;
```

Test a query:

```sql
SELECT * FROM students;
```

If everything is configured correctly, the database is ready for your backend application.

---

# 8. Exit MariaDB

When finished, exit the MariaDB shell:

```sql
EXIT;
```

---

# Production Security Recommendations

For production deployments, follow these practices:

### 1. Never use the root database account

Create a dedicated application user:

```sql
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD';
```

Grant access only to the required database:

```sql
GRANT ALL PRIVILEGES ON student_db.* TO 'app_user'@'localhost';
```

---

### 2. Use strong passwords

Use a long, randomly generated password.

Avoid passwords such as:

```text
password
admin123
root123
student123
```

---

### 3. Never commit secrets

Do not put credentials directly into source code or commit them to Git:

```properties
spring.datasource.password=myRealPassword
```

Instead, use environment variables:

```properties
spring.datasource.password=${DB_PASS}
```

---

### 4. Restrict database access

MariaDB normally listens on port:

```text
3306
```

Do not expose port `3306` publicly unless remote database access is explicitly required.

If the backend and MariaDB are running on the same server, prefer local/private communication.

---

### 5. Keep MariaDB updated

Regularly update your operating system and database packages:

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Quick Setup

For a fresh Ubuntu server, the basic setup is:

```bash
sudo apt update
sudo apt install mariadb-server -y

sudo systemctl enable mariadb
sudo systemctl start mariadb

sudo mysql_secure_installation
```

Then:

```bash
sudo mariadb
```

Inside MariaDB:

```sql
CREATE DATABASE student_db;

CREATE USER 'app_user'@'localhost'
IDENTIFIED BY 'YOUR_STRONG_PASSWORD';

GRANT ALL PRIVILEGES ON student_db.* 
TO 'app_user'@'localhost';

FLUSH PRIVILEGES;

USE student_db;
```

Create the required tables and then exit:

```sql
EXIT;
```

---

## Database Connection Checklist

Before connecting your backend, verify:

```text
☑ MariaDB is installed
☑ MariaDB service is running
☑ student_db exists
☑ Application database user exists
☑ User has privileges on student_db
☑ DB_HOST is correct
☑ DB_PORT is 3306
☑ DB_USER is correct
☑ DB_PASS is correct
☑ DB_NAME is student_db
☑ Port 3306 is reachable when remote access is required
☑ Database credentials are stored securely
```

---

## Troubleshooting

### Check MariaDB status

```bash
sudo systemctl status mariadb
```

### Restart MariaDB

```bash
sudo systemctl restart mariadb
```

### Check whether MariaDB is listening on port 3306

```bash
sudo ss -lntp | grep 3306
```

### Test database login

```bash
mysql -u app_user -p -h localhost -P 3306
```

### Check databases

```sql
SHOW DATABASES;
```

### Check users

```sql
SELECT User, Host FROM mysql.user;
```

### Check application privileges

```sql
SHOW GRANTS FOR 'app_user'@'localhost';
```

---

## Architecture

The recommended application architecture is:

```text
                    ┌──────────────────┐
                    │   Backend App    │
                    └────────┬─────────┘
                             │
                       JDBC / TCP
                             │
                    ┌────────▼─────────┐
                    │     MariaDB      │
                    │     :3306        │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │   student_db     │
                    └──────────────────┘
```

The backend should use a **dedicated database user** and should never require the MariaDB root credentials.

---



## Maintainer

**Project:** Student Registration / EasyCRUD Database Setup

For issues or improvements, please open an issue or submit a pull request in the project repository.
