# Library Management System using SQL 

## Project Overview

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase my database design, manipulation, and querying skills.

![image](https://github.com/user-attachments/assets/2886ab46-9178-4307-b0a7-75edeadcbde2)

## Objectives

1. Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
3. CTE (Common Table Expression): Utilize CTE to create a temporary table based on query results.
4. Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

## Project Workflow

## 1. Database Setup

- Database Creation: Created a database named library_management_system.
- Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.
```
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);

-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(30) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);

-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);

-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id)
);

-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

## CRUD Operations

- Create: Inserted sample records into the books table.
- Read: Retrieved and displayed data from various tables.
- Update: Updated records in the employees table.
- Delete: Removed records from the members table as needed.

### Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```

### Task 2: Update an Existing Member's Address
```
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
SELECT * FROM members;
```

### Task 3: Delete a Record from the Issued Status Table  -- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```
SELECT * FROM issued_status
WHERE issued_id = 'IS121';
```
```
DELETE FROM issued_status
WHERE issued_id = 'IS121'
```

### Task 4: Retrieve All Books Issued by a Specific Employee  -- Objective: Select all books issued by the employee with emp_id = 'E101'.
```
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

### Task 5: List Members Who Have Issued More Than One Book  -- Objective: Use GROUP BY to find members who have issued more than one book.
```
SELECT
    issued_emp_id,
    COUNT(*) as no_of_books
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(*) > 1
```

## Data Analysis & Findings

The following SQL queries were used to address specific questions:

### Task 6: Retrieve All Books in a Specific Category
```
SELECT * FROM books
WHERE category = 'Classic';
```

### Task 7: Find Total Rental Income by Category:
```
SELECT 
    b.category,
    SUM(b.rental_price) AS total_rental_income,
    COUNT(ist.issued_id) AS total_rentals
FROM 
    issued_status AS ist
JOIN
    books AS b
ON 
    b.isbn = ist.issued_book_isbn
GROUP BY 
    b.category;
```

### Task 8: List Members Who Registered in the Last 180 Days:
```
SELECT * FROM members
WHERE reg_date >= GETDATE() - 180;
```

### Task 9: List Employees with Their Branch Manager's Name and their branch details:
```
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
FROM employees as e1
JOIN 
branch as b
ON e1.branch_id = b.branch_id    
JOIN
employees as e2
ON e2.emp_id = b.manager_id
```

### Task 10: Retrieve the List of Books Not Yet Returned
```
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

### Task 11:  Identify Members with Overdue Books
-- Write a query to identify members who have overdue books (assume a 30-day return period). 
-- Display the member's_id, member's name, book title, issue date, and days overdue.
```
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    -- rs.return_date,
    DATEDIFF(DAY, ist.issued_date, GETDATE()) AS over_dues_days
FROM 
    issued_status AS ist
JOIN 
    members AS m
    ON m.member_id = ist.issued_member_id
JOIN 
    books AS bk
    ON bk.isbn = ist.issued_book_isbn
LEFT JOIN 
    return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND 
    DATEDIFF(DAY, ist.issued_date, GETDATE()) > 30
ORDER BY 
    ist.issued_member_id;

GO;
```

### Task 12: Update Book Status on Return
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).
```
CREATE OR ALTER PROCEDURE add_return_records
    @p_return_id VARCHAR(10),
    @p_issued_id VARCHAR(10),
    @p_book_quality VARCHAR(10)
AS
BEGIN
    DECLARE 
        @v_isbn VARCHAR(50),
        @v_book_name VARCHAR(80);

    -- Insert into return_status
  INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
  VALUES (@p_return_id, @p_issued_id, GETDATE(), @p_book_quality);

    -- Retrieve the book details and assign to variables
  SELECT 
      @v_isbn = issued_book_isbn,
      @v_book_name = issued_book_name
  FROM 
      issued_status
  WHERE 
      issued_id = @p_issued_id;

    -- Update book status to 'yes'
  UPDATE books
  SET status = 'yes'
  WHERE isbn = @v_isbn;

    -- Output message
  PRINT 'Thank you for returning the book: ' + @v_book_name;
END;

-- Test call:
EXEC add_return_records 'RS138', 'IS135', 'Good';
EXEC add_return_records 'RS148', 'IS140', 'Good';
```

### Task 13: Branch Performance Report
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.
```
WITH BranchPerformance AS (
    SELECT 
        b.branch_id,
        b.manager_id,
        COUNT(ist.issued_id) AS number_book_issued,
        COUNT(rs.return_id) AS number_of_book_return,
        SUM(bk.rental_price) AS total_revenue
    FROM 
        issued_status AS ist
    JOIN 
        employees AS e
        ON e.emp_id = ist.issued_emp_id
    JOIN 
        branch AS b
        ON e.branch_id = b.branch_id
    LEFT JOIN 
        return_status AS rs
        ON rs.issued_id = ist.issued_id
    JOIN 
        books AS bk
        ON ist.issued_book_isbn = bk.isbn
    GROUP BY 
        b.branch_id, b.manager_id
)SELECT * FROM BranchPerformance;
```

###  Task 14: Create a Table of Active Members
Use the CTE statement to create a new table active_members containing members who have issued at least one book in the last 2 months.
```
WITH ActiveMembersCTE AS (
    SELECT * FROM members
WHERE member_id IN (
	SELECT 
	DISTINCT issued_member_id   
	FROM issued_status
	WHERE DATEDIFF(MONTH, issued_date, GETDATE()) <= 2
	)
)SELECT * FROM ActiveMembersCTE;
```

## -- Task 15: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. 
-- Display the employee name, number of books processed, and their branch.
```
   SELECT TOP 3
        e.emp_name AS emp_name,
        b.branch_id AS branch_id,  -- Assuming `branch_name` is a column in the `branch` table
        COUNT(ist.issued_id) AS no_book_issued
    FROM issued_status AS ist
    JOIN employees AS e
        ON e.emp_id = ist.issued_emp_id
    JOIN branch AS b
        ON e.branch_id = b.branch_id
    GROUP BY e.emp_name, b.branch_id
	ORDER BY COUNT(ist.issued_id) DESC

GO;
```

## -- Task 16: Stored Procedure Objective: Create a stored procedure to manage the status of books in a library system. 
Description: Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows: The stored procedure should take the book_id as an input parameter. The procedure should first check if the book is available (status = 'yes'). If the book is available, it should be issued, and the status in the books table should be updated to 'no'. If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.
```
CREATE PROCEDURE issue_book
    @p_issued_id NVARCHAR(10),
    @p_issued_member_id NVARCHAR(30),
    @p_issued_book_isbn NVARCHAR(30),
    @p_issued_emp_id NVARCHAR(10)
AS
BEGIN
    DECLARE @v_status NVARCHAR(10);

    -- Checking if book is available 'yes'
  SELECT @v_status = status
    FROM books
    WHERE isbn = @p_issued_book_isbn;

  IF @v_status = 'yes'
    BEGIN
        -- Insert into issued_status
        INSERT INTO issued_status (issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (@p_issued_id, @p_issued_member_id, GETDATE(), @p_issued_book_isbn, @p_issued_emp_id);

        -- Update the book status
  UPDATE books
        SET status = 'no'
        WHERE isbn = @p_issued_book_isbn;

  PRINT 'Book records added successfully for book ISBN: ' + @p_issued_book_isbn;
  END
  ELSE
  BEGIN
        PRINT 'Sorry to inform you, the book you have requested is unavailable. Book ISBN: ' + @p_issued_book_isbn;
    END
END;

-- Checking initial data
SELECT * FROM books;
SELECT * FROM issued_status;

-- Calling the procedure
EXEC issue_book 'IS155', 'C108', '978-0-553-29698-2', 'E104';
EXEC issue_book 'IS156', 'C108', '978-0-375-41398-8', 'E104';

-- Checking the updated data
SELECT * FROM books WHERE isbn = '978-0-375-41398-8';
```

## Conclusion
This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
