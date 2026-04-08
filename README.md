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

  ![Alt Text](https://github.com/ragi-krish/Library_System_Management_Using_SQL/blob/main/erd.png)

## 🛠️ Key Features & SQL Techniques
1. Database Implementation (DDL)
2. Analytical Queries & CRUD
3. Advanced Automation
4. CTAS (Create Table As Select)
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
```sql
SELECT 
    m.member_id,
    m.member_name,
    b.book_title,
    ist.issued_date,
    -- Calculate days overdue: (Today - Issue Date) - 30 days
    (CURRENT_DATE - ist.issued_date) - 30 AS days_overdue
FROM issued_status as ist
JOIN members as m 
    ON m.member_id = ist.issued_member_id
JOIN books as b 
    ON ist.issued_book_isbn = b.isbn
LEFT JOIN return_status as rs 
    ON ist.issued_id = rs.issued_id
WHERE 
    rs.return_id IS NULL -- Book has not been returned
    AND 
    (CURRENT_DATE - ist.issued_date) > 30
	ORDER BY member_id ; -- More than 30 days have passed
```
#### Update Book Status on Return: Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).
```sql
create or replace procedure add_value_return(p_return_id varchar(8),p_issued_id varchar(5))
language plpgsql
as $$
declare 
	var_isbn varchar(25);
	var_book_name varchar(80);
begin
	insert into return_status(return_id,issued_id,return_date)
	values(p_return_id,p_issued_id,current_date);

	select  issued_book_isbn,
        	issued_book_name
		into
		var_isbn 
		var_book_name 
	from issued_status
	where issued_id = p_issued_id;

	update books
	set status = 'yes'
	where isbn = var_isbn;
	
end;
$$;

call add_value_return('RS119','IS135')
```
#### Branch Performance Report: Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.
```sql
select br.branch_id,
	br.branch_address,
	b.book_title,
	sum(b.rental_price) as total_revenue,
	count(i.issued_id) as number_of_books_issued,
	count(r.return_id) as number_of_books_returned
	
from issued_status as i
join employee as e
on i.issued_emp_id = e.emp_id
join branch as br
on br.branch_id = e.branch_id
left join return_status as r
on r.issued_id = i.issued_id
join books as b
on b.isbn = i.issued_book_isbn
group by 1,3
order by 1;
```
#### Create a Table of active_members containing members who have issued at least one book in the last 2 months.
```sql
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (SELECT 
                        DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE 
                        issued_date >= CURRENT_DATE - INTERVAL '2 month'
                    );

SELECT * FROM active_members;
```
#### Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch details.
```sql
SELECT 
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist
JOIN
employee as e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2
ORDER BY no_book_issued DESC LIMIT 3;
```
#### Stored Procedure Objective: Create a stored procedure to manage the status of books in a library system. Description: Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows: The stored procedure should take the book_id as an input parameter. The procedure should first check if the book is available (status = 'yes'). If the book is available, it should be issued, and the status in the books table should be updated to 'no'.If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.
```sql
create or replace procedure book_issue_check
								(p_issued_id VARCHAR(10),
								p_member_id VARCHAR(30),
								p_book_isbn VARCHAR(30),
								p_issued_emp_id VARCHAR(10))								
language plpgsql
as $$
declare
	v_status varchar(10);
	v_book_name varchar(100);

begin
	select status,book_title 
	into v_status,v_book_name
	from books 
	where isbn = p_book_isbn;

	if v_status = 'yes' then
		insert into issued_status(issued_id, issued_member_id,issued_book_name, issued_date, issued_book_isbn, issued_emp_id)
		values(p_issued_id,p_member_id,v_book_name,current_date,p_book_isbn,p_issued_emp_id);

		update books set status='no'
		where isbn = p_book_isbn;
		
	else
		raise notice 'the requested book % is not available',p_book_isbn;
	end if;
end;
$$

SELECT * FROM books;
SELECT * FROM issued_status;

call book_issue_check('IS156', 'C108', '978-0-330-25864-8', 'E104');
call book_issue_check('IS157', 'C108', '978-0-375-41398-8', 'E104');
```
#### Create Table As Select (CTAS) Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines. Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include: The number of overdue books. The total fines, with each day's fine calculated at $0.50. The number of books issued by each member. The resulting table should show: Member ID Number of overdue books Total fines.
```sql
create table fine_table as(
select issued_member_id, count(issued_book_isbn) as overdue_books, 
sum(
	(case
		when r.return_date is not null
		then 
			(r.return_date-i.issued_date-30)
		else
			(current_date-i.issued_date-30)
		end
	 )*.5
    ) as fine
from issued_status as i
left join return_status as r
on i.issued_id = r.issued_id
where (r.return_date-i.issued_date) > 30 -- case 1
or 
(r.return_id is null) and (current_date-i.issued_date) >30 
group by issued_member_id);


select * from fine_table;
```
## Reports
* Database Schema: Detailed table structures and relationships.
* Data Analysis: Insights into book categories, employee salaries, member registration trends, and issued books.
* Summary Reports: Aggregated data on high-demand books and employee performance.
## 🏁 Conclusion
This Library Management System project successfully demonstrates the transition from raw data to a functional, automated relational database. By implementing this system, I have addressed several core challenges in data m.anagement such as Integrity & Consistency,Operational Efficiency, Data-Driven Insights,Proactive Management.
