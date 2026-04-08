# Library_System_Management_Using_SQL
A comprehensive SQL-based library management system demonstrating data modeling, CRUD operations, advanced joins, and automation via stored procedures.

## 📊 Database Schema
The system consists of the following relational tables:

* Branch: Information about library branches and their managers.

* Employee: Staff details associated with specific branches.

* Members: Registered library patrons.

* Books: Inventory details including ISBN, category, rental price, and status.

* Issued_Status: Tracks books currently checked out by members.

* Return_Status: Records returned books and links back to issuance records.

## 🛠️ Key Features & SQL Techniques
1. Database Implementation (DDL)
Standardized table creation with defined Primary Keys and Foreign Keys to ensure data integrity.

2. Analytical Queries & CRUD
Data Updates: Dynamic updates for member information.

Date Arithmetic: Identifying members registered in the last 180 days using CURRENT_DATE - INTERVAL.

Aggregation: Reporting on rental income per category and book using GROUP BY and SUM/COUNT.

Complex Joins: Generating branch performance reports by joining five distinct tables to calculate revenue and volume.

3. Advanced Automation
The project includes PostgreSQL Stored Procedures to handle business logic:

add_value_return: Automatically updates book status to 'yes' (available) when a record is added to the return table.

book_issue_check: Validates book availability before issuance. If a book is already out, it prevents the transaction and raises a custom notice.

4. CTAS (Create Table As Select)
Used for data snapshots and reporting:

Active Members: Identifies high-engagement users from the last 2 months.

Financial Reporting: Creates tables for expensive inventory and overdue fine calculations.
