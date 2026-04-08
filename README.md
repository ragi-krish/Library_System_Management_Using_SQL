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
### CRUD OPERATIONS
#### Update an Existing Member's Address
```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```
#### Delete a Record from the Issued Status Table
```sql
DELETE FROM issued_status
WHERE  issued_id =   'IS121';
```
#### List Members Who Registered in the Last 180 Days
```sql
select member_id from members
where reg_date >= current_date - interval '180 days';
```
#### Retrieve All Books in a Specific Category
```sql
SELECT * FROM books WHERE category = 'Classic';
```
#### Create a summary table of books and no: of times issued
```sql
create table book_issued_count as 
select isbn, book_title,count(issued_book_isbn) as issued_count 
from books as b 
left join issued_status as i 
on b.isbn = i.issued_book_isbn 
group by b.isbn,b.book_title;
```
#### find no: of books in each category
```sql
select count(isbn), category from books
group by category;
```
#### Find Total Rental Income by book
```sql
select 
issued_book_name,
issued_book_isbn,
rental_price,
count(issued_id) as no_of_times_issued,
count(issued_id)*rental_price as total_rent
from issued_status as i join 
books as b on 
i.issued_book_isbn = b.isbn
group by issued_book_name,issued_book_isbn,rental_price;
```
#### Find Total Rental Income by Category
```sql
select 
category,
count(issued_id) as no_of_times_issued,
sum(rental_price)
from issued_status as i join 
books as b on 
i.issued_book_isbn = b.isbn
group by category;
```
#### List Employees with Their Branch Manager's Name and their branch details
```sql
select e.emp_id,e.emp_name,
b.* ,
e2.emp_name as manager_name
from employee as e
right join branch as b
on b.branch_id = e.branch_id
join employee as e2
on b.manager_id = e2.emp_id;
```
#### Create a Table of Books with Rental Price Above a Certain Threshold
```sql
create table expensive_books as 
select isbn,book_title,rental_price
from books where rental_price >6;
```
#### Retrieve the List of Books Not Yet Returned
```sql
select i.issued_book_name, i.issued_date,
r.return_date
from issued_status as i
left join return_status as r
on i.issued_id = r.issued_id
where return_id is NULL;
```
#### Identify Members with Overdue Books: Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.



