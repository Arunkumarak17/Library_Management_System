# Library_Management_System

## Project Overview

**Project Title**: Library_Management_System

This project is a Library Management System developed using SQL Server to handle core library operations such as managing books, members, issuing and returning books, and tracking overdue items. It focuses on creating a structured relational database with tables like books, members, issued_status, and return_status, connected through primary and foreign keys for data integrity.

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
**Database Creation**: Created a database named `library_db`.
**Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

**use the existing database**

use SQL_Project
```SQL
create table books (
	isbn varchar(100) primary key,
	book_title varchar(100),
	category varchar(100),
	rental_price decimal(10,2),
	status varchar(100),
	author varchar(100),
	publisher varchar(100));

select * from books


create table branch(
	branch_id varchar(100) primary key,
	manager_id varchar(100),
	branch_address varchar(100),
	contact_no varchar(100));


select * from branch

create table employees(
	emp_id varchar(100) primary key,
	emp_name varchar (100),
	position varchar(100),
	salary int,
	branch_id varchar(100));

select * from employees

create table issued_status(
	issued_id varchar(100) primary key,
	issued_member_id varchar(100),
	issued_book_name varchar(100),
	issued_date date,
	issued_book_isbn varchar(100),
	issued_emp_id varchar(100));


create table members(
	member_id varchar(100) primary key,
	member_name varchar(100),
	member_address varchar(100),
	reg_date date);


create table return_status(
	return_id varchar(100),
	issued_id varchar(100),
	return_book_name varchar(100),
	return_date date,
	return_book_isbn varchar(100));
```
#Connecting the tables using foreign key constraint

```SQL

alter table issued_status
add constraint fk_members
foreign key (issued_member_id)
references members(member_id)

alter table issued_status
add constraint fk_books
foreign key (issued_book_isbn)
references books(isbn)

alter table issued_status
add constraint fk_employees
foreign key (issued_emp_id)
references employees(emp_id)

alter table employees
add constraint fk_branch
foreign key (branch_id)
references branch(branch_id)


alter table return_status
add constraint fk_issued_status
foreign key (issued_id)
references issued_status(issued_id)

select * from issued_status
select * from return_status

SELECT rs.issued_id
FROM dbo.return_status rs
LEFT JOIN dbo.issued_status isd
    ON rs.issued_id = isd.issued_id
WHERE isd.issued_id IS NULL
  AND rs.issued_id IS NOT NULL;


DELETE rs
FROM dbo.return_status rs
LEFT JOIN dbo.issued_status isd
    ON rs.issued_id = isd.issued_id
WHERE isd.issued_id IS NULL;


UPDATE dbo.return_status
SET issued_id = (SELECT TOP 1 issued_id FROM dbo.issued_status)
WHERE issued_id NOT IN (SELECT issued_id FROM dbo.issued_status);

INSERT INTO dbo.issued_status (issued_id)
SELECT DISTINCT issued_id
FROM dbo.return_status
WHERE issued_id NOT IN (SELECT issued_id FROM dbo.issued_status);

ALTER TABLE dbo.return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES dbo.issued_status(issued_id);


SELECT * FROM books;
SELECT * FROM branch;
SELECT * FROM employees;
SELECT * FROM issued_status;
SELECT * FROM return_status;
SELECT * FROM members;

```

## Project TASK

### 2. CRUD Operations

 **Create**: Inserted sample records into the `books` table.
 **Read**: Retrieved and displayed data from various tables.
 **Update**: Updated records in the `employees` table.
 **Delete**: Removed records from the `members` table as needed.


** Task 1. Create a New Book Record
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```SQL
insert into books(isbn,book_title,category,rental_price,status,author,publisher)
values( '978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')
select * from books
```

-- Task 2: Update an Existing Member's Address
```SQL
update members
set member_address='125 main st'
where member_id='C101'
select * from members

```
-- Task 3: Delete a Record from the Issued Status Table
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```SQL
delete from issued_status
where issued_id='IS121'
select * from issued_status
where issued_id='IS121'

```

-- Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'.

```SQL
select * from issued_status
where issued_emp_id='E101'
```

-- Task 5: List Members Who Have Issued More Than One Book -- Objective: Use GROUP BY to find members who have issued more than one book.
```SQL
select issued_emp_id,COUNT(issued_id) as total_book_issued
from issued_status
group by issued_emp_id
having COUNT(issued_id) >1

```
-- CTAS
-- Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```SQL

Select b.isbn,b.book_title,COUNT(ist.issued_id) as no_issued
into book_cnts
from books as b
join 
issued_status as ist
on ist.issued_book_isbn=b.isbn
group by b.isbn,b.book_title

select * from book_cnts
```
-- Task 7. Retrieve All Books in a Specific Category:

```SQL
SELECT * FROM books
WHERE category = 'Classic'
```

-- Task 8: Find Total Rental Income by Category:
```SQL
select b.category,sum(b.rental_price) as Total_price, count(*) as no_issued
from books as b
join issued_status as ist
on ist.issued_book_isbn=b.isbn
group by b.category
```
-- List Members Who Registered in the Last 180 Days:

```SQL
select * from members
where reg_date>= DATEADD(day,-180, GETDATE())



insert into members(member_id,member_name,member_address,reg_date)
values ('C128','Arun','143 marraige','2026-01-14'),
('C129','Anajana','143 love','2026-01-15')

```
-- task 10 List Employees with Their Branch Manager's Name and their branch details:
```SQL
select * from employees
select * from branch

select e1.*,
	   b.manager_id,
	   e2.emp_name as manager
from employees as e1
join branch as b
on b.branch_id=e1.branch_id
join employees as e2
on b.manager_id=e2.emp_id
```

-- Task 11. Create a Table of Books with Rental Price Above a Certain Threshold 8USD:

```SQL
drop table if exists book_sales

select * 
into book_sales
from books
where rental_price > 8

```

-- Task 12: Retrieve the List of Books Not Yet Returned
```SQL
select distinct ist.issued_book_name
from issued_status as ist
left join return_status as rs
on ist.issued_id=rs.issued_id
where rs.return_id is null
```
/*
Task 13: 
Identify Members with Overdue Books
Write a query to identify members who have overdue books (assume a 30-day return period).
Display the member's_id, member's name, book title, issue date, and days overdue.


-- issued_status == members == books == return_status
-- filter books which is return
-- overdue > 500
*/

```SQL
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    DATEDIFF(DAY, ist.issued_date, GETDATE()) AS over_dues_days
FROM issued_status AS ist
JOIN members AS m
    ON m.member_id = ist.issued_member_id
JOIN books AS bk
    ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND DATEDIFF(DAY, ist.issued_date, GETDATE()) > 30
ORDER BY ist.issued_member_id;

```
-- 
/*    
Task 14: Update Book Status on Return
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).
*/

```SQL

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-330-25864-8';
-- IS104

SELECT * FROM books
WHERE isbn = '978-0-451-52994-2';

UPDATE books
SET status = 'no'
WHERE isbn = '978-0-451-52994-2';

SELECT * FROM return_status
WHERE issued_id = 'IS130';

ALTER TABLE return_status
ADD book_quality VARCHAR(20);


-- 
INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
VALUES ('RS125', 'IS130', CAST(GETDATE() AS DATE), 'Good');

SELECT * 
FROM return_status
WHERE issued_id = 'IS130';



CREATE OR ALTER PROCEDURE add_return_records
    @p_return_id VARCHAR(10),
    @p_issued_id VARCHAR(10),
    @p_book_quality VARCHAR(10)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE 
        @v_isbn VARCHAR(50),
        @v_book_name VARCHAR(80);

    -- Insert into return_status
    INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
    VALUES (@p_return_id, @p_issued_id, CAST(GETDATE() AS DATE), @p_book_quality);

    -- Get book details
    SELECT 
        @v_isbn = issued_book_isbn,
        @v_book_name = issued_book_name
    FROM issued_status
    WHERE issued_id = @p_issued_id;

    -- Update book status
    UPDATE books
    SET status = 'yes'
    WHERE isbn = @v_isbn;

    -- Message
    PRINT 'Thank you for returning the book: ' + ISNULL(@v_book_name,'');
END;
GO

    



-- Testing FUNCTION add_return_records

SELECT * 
FROM issued_status 
WHERE issued_id = 'IS135';

SELECT * 
FROM books 
WHERE isbn = '978-0-307-58837-1';

SELECT * 
FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * 
FROM return_status
WHERE issued_id = 'IS135';


-- calling function 
EXEC add_return_records 'RS138', 'IS135', 'Good';




-- calling function 
EXEC add_return_records 'RS148', 'IS140', 'Good';

```

#Conclusion:
The Library Management System project demonstrates the effective use of SQL for database design, data management, and query optimization. It streamlines book tracking, member management, and issue/return processes while ensuring data accuracy and efficiency.This project highlights practical skills in database development, problem-solving, and structured data analysis.

# Author Arunkumar
