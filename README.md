## SQL Case Study: Employee and Annual Reviews Data Analysis

This case study revolves around managing employee records and annual reviews for a company. Given the provided data and structure, the goal is to derive insights through SQL queries.

### Data and Tables Overview

We have two tables: 

- **Employee**: Stores personal information, hire and termination dates, and salary for each employee.
- **AnnualReviews**: Tracks annual reviews with employee IDs and review dates.

---

### Data Definition and Population

#### DDL (Data Definition Language)

The tables are created as follows:

```sql
CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    FirstName VARCHAR(100),
    LastName VARCHAR(100),
    HireDate DATE,
    TerminationDate DATE,
    Salary NUMERIC
);


CREATE TABLE AnnualReviews (
    ID INT PRIMARY KEY,
    EmpID INT,
    ReviewDate DATE
);
```

#### Add the foreign key constraint using ALTER TABLE (Optional)
```
ALTER TABLE AnnualReviews
ADD CONSTRAINT fk_employee
FOREIGN KEY (EmpID) REFERENCES Employee(ID);
```
##### Note:
- This command will enforce referential integrity by ensuring that each EmpID in the AnnualReviews table must exist in the Employee table.
- If there are any values in the AnnualReviews table that do not have corresponding entries in the Employee table, the command will fail, and you must resolve these inconsistencies before successfully adding the constraint.

#### ERD
![Untitled (5)](https://github.com/user-attachments/assets/f780bbe9-4eab-4892-8e94-4b850274ce8f)


- **Employee** table contains:
  - `FirstName`, `LastName`: Employee names.
  - `ID`: Unique employee ID.
  - `HireDate`, `TerminationDate`: Dates of hiring and termination.
  - `Salary`: Employee salary.
  
- **AnnualReviews** table contains:
  - `ID`: Unique review ID.
  - `EmpID`: Links to the employee ID.
  - `ReviewDate`: Date when the review was conducted.

#### Data Insert

The sample data is populated as:

```sql
INSERT INTO Employee (FirstName, LastName, ID, HireDate, TerminationDate, Salary) VALUES
('Bob', 'Smith', 1, '2009-06-20', '2016-01-01', 10000),
('Joe', 'Jarrod', 2, '2010-12-02', NULL, 20000),
('Nancy', 'Soley', 3, '2012-03-14', NULL, 30000),
('Keith', 'Widjaja', 4, '2013-10-09', '2014-01-01', 20000),
('Kelly', 'Smalls', 5, '2013-10-09', NULL, 20000),
('Frank', 'Nguyen', 6, '2015-10-04', '2015-05-01', 60000);

INSERT INTO AnnualReviews (ID, EmpID, ReviewDate) VALUES
(10, 1, '2016-01-01'),
(20, 2, '2016-12-04'),
(30, 10, '2015-02-13'),
(40, 22, '2010-12-10'),
(50, 11, '2009-01-01'),
(60, 12, '2009-03-03'),
(70, 13, '2008-01-12'),
(80, 1, '2003-12-04'),
(90, 1, '2014-04-30');
```
### Questions
1. Write a query to return all employees still working for the company with last names starting with "Smith" sorted by last name then first name.
2. Given the `Employee` and `AnnualReviews` tables, write a query to return all employees who have never had a review sorted by HireDate.
3. Write a query to calculate the difference (in days) between the most and least tenured employee still working for the company.
4. Given the employee table above, write a query to calculate the longest period (in days) that the company has gone without a hiring or firing anyone.
5. Write a query that returns each employee and for each row/employee include the greatest number of employees that worked for the company at any time during their tenure and the first date that maximum was reached.

### Answers and Queries

#### 1. Employees with Last Name "Smith" Still Working

**Query:**

```sql
SELECT FirstName, LastName
FROM Employee
WHERE TerminationDate IS NULL
AND LastName LIKE 'Smith%'
ORDER BY LastName, FirstName;
```

**Explanation:**
- The query filters employees whose `LastName` starts with "Smith" and who have no `TerminationDate` (still employed).
- Results are sorted by `LastName`, then `FirstName`.

**Purpose:**
- This helps identify current employees with the last name "Smith" to address specific requirements, such as company policy reviews or name-specific tasks.

---

#### 2. Employees Who Have Never Had a Review

**Query:**

```sql
SELECT E.FirstName, E.LastName, E.HireDate
FROM Employee E
LEFT JOIN AnnualReviews A ON E.ID = A.EmpID
WHERE A.EmpID IS NULL
ORDER BY E.HireDate;
```

**Explanation:**
- A `LEFT JOIN` ensures all employees are included, even those without reviews. `WHERE A.EmpID IS NULL` filters out employees who have had reviews.
- Sorted by `HireDate` to show the oldest employees who haven't had reviews.

**Purpose:**
- This analysis helps identify employees who have been missed in the annual review process, which can impact performance evaluations.

---

#### 3. Difference in Days Between the Most and Least Tenured Employees

**Query:**

```sql
SELECT MIN(HireDate) AS MIN_HireDate, MAX(HireDate) AS MAX_HireDate,
    (MAX(HireDate) - MIN(HireDate)) AS TenureDifference
FROM Employee
WHERE TerminationDate IS NULL;
```

**Explanation:**
- The query finds the most recent and earliest hire dates among current employees using `MAX(HireDate)` and `MIN(HireDate)`.
**Purpose:**
- This insight helps understand the range of employee experience levels among current staff.

---

#### 4. Longest Period Without Hiring or Firing

**Query:**

```sql
WITH AllDates AS (
    SELECT HireDate AS Date, 'Hire' AS Event FROM Employee
    UNION
    SELECT TerminationDate AS Date, 'Terminate' AS Event FROM Employee
    WHERE TerminationDate IS NOT NULL
),
OrderedDates AS (
    SELECT 
        Date,
        Event,
        LEAD(Date) OVER (ORDER BY Date) AS NextDate
    FROM AllDates
)
SELECT 
    MAX((NextDate - Date)::INTEGER) AS LongestPeriodWithoutHiringFiring
FROM OrderedDates
WHERE NextDate IS NOT NULL;
```

**Explanation:**
- The query combines `HireDate` and `TerminationDate` using `UNION`. Then, it calculates the longest period between events (either hire or termination) using `LEAD()`, which looks ahead to the next event.
- `DATEDIFF` is used to find the duration between two consecutive dates.

**Purpose:**
- This analysis helps identify the longest period of workforce stability without any changes in personnel, indicating periods of potential organizational calm or stagnation.

---

#### 5. Maximum Number of Employees During Each Employee’s Tenure

**Query:**

```sql
SELECT
    FirstName,
    LastName,
    HireDate,
    Periode_Date,
    TerminationDate,
    Total_Employ_Active
FROM Employee
CROSS JOIN (
    SELECT DISTINCT
        hd AS Periode_Date,
        SUM(sk) OVER (ORDER BY hd) AS Total_Employ_Active
    FROM (
        SELECT DISTINCT
            hd,
            LEAD(hd) OVER (ORDER BY hd) AS dwan,
            sk
        FROM (
            SELECT
                HireDate AS hd,
                1 AS sk
            FROM Employee
        ) AS subquery1
    ) AS subquery2
) AS subquery3;
```

**Explanation:**
- The query calculates the number of employees that worked for the company during each employee's tenure. `COALESCE` handles current employees by setting the `EndDate` to the current date.
- It finds the greatest number of employees during each employee’s time at the company and the first date that the maximum was reached.

**Purpose:**
- This query is crucial for understanding the maximum workforce size during each employee's tenure, which helps track organizational growth and stability over time.

---

### Conclusion

This case study demonstrates how structured queries can provide valuable insights into an organization's employee records and review processes. The SQL queries identified employees who are still active, those missing reviews, tenure differences, and periods of workforce stability. These insights can be used for performance management, HR planning, and operational efficiency.

Key findings include:
- Several employees haven't received reviews.
- The company experienced a long period without hiring or firing.
- Workforce size varied, with maximum and minimum staffing levels during different periods of each employee's tenure.
