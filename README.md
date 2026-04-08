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

* Date Arithmetic: Identifying members registered in the last 180 days using CURRENT_DATE - INTERVAL.

* Aggregation: Reporting on rental income per category and book using GROUP BY and SUM/COUNT.

* Complex Joins: Generating branch performance reports by joining five distinct tables to calculate revenue and volume.

3. Advanced Automation
The project includes PostgreSQL Stored Procedures to handle business logic:

* add_value_return: Automatically updates book status to 'yes' (available) when a record is added to the return table.

* book_issue_check: Validates book availability before issuance. If a book is already out, it prevents the transaction and raises a custom notice.

4. CTAS (Create Table As Select)
Used for data snapshots and reporting:

* Active Members: Identifies high-engagement users from the last 2 months.

* Financial Reporting: Creates tables for expensive inventory and overdue fine calculations.

## Queries
```sql
--Create Table Employee
drop table if exists employee;
create table employee
	(emp_id varchar(5) PRIMARY KEY,	
	emp_name varchar(20),
	position varchar(10),	
	salary int,
	branch_id varchar(5),
	FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
	);

--Create Table members
drop table if exists members
create table members
	(member_id varchar(5) PRIMARY KEY,	
	member_name varchar(15),
	member_address varchar(15),	
	reg_date date
	)

--Create Table branch
drop table if exists branch
create table branch
	(branch_id varchar(5) PRIMARY KEY,	
	manager_id varchar(5),
	branch_address varchar(15),	
	contact_no varchar(15)
	)

--Create Table books
drop table if exists books
create table books
	(isbn varchar(20) PRIMARY KEY,	
	book_title varchar(50),
	category varchar(20),	
	rental_price float,
	status varchar(5),
	author	varchar(20),
	publisher varchar(25)
	)

--Create Table issued_status
drop table if exists issued_status
create table issued_status
	(issued_id varchar(8) PRIMARY KEY,	
	issued_member_id varchar(5),
	issued_book_name varchar(30),	
	issued_date date,
	issued_book_isbn varchar(25),
	issued_emp_id	varchar(5),
	FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employee(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
	)

--Create Table return_status
drop table if exists return_status
create table return_status
	(return_id varchar(8) PRIMARY KEY,	
	issued_id varchar(5),
	return_book_name varchar(30),	
	return_date date,
	return_book_isbn varchar(25),
	FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
	)
```
