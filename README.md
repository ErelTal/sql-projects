# sql project - Employees database

-- **Using Joins** --
-- **Assignment** -> Join the 'employees' and the 'dept_manager' tables to return a subset of all the employees whose last name is Markovitch --

# Solution
SELECT
    e.emp_no,  
    e.first_name,  
    e.last_name,  
    dm.dept_no,  
    dm.from_date  
FROM  
    employees e  
        LEFT JOIN   
dept_manager dm ON e.emp_no = dm.emp_no  
WHERE  
    e.last_name = 'Markovitch'  
ORDER BY dm.dept_no DESC, e.emp_no;



-- **Using Joins and Where together**--
-- **Assignment** -> Select the first and last name, the hire date, and the job title of all employees whose first name is “Margareta” and have the last name “Markovitch”. --

# Solution
SELECT
    e.first_name, e.last_name, e.hire_date, t.title
FROM
    employees e
        JOIN
    titles t ON e.emp_no = t.emp_no
WHERE
    first_name = 'Margareta'
        AND last_name = 'Markovitch'
ORDER BY e.emp_no;  


-- **Using Unions** --
-- **Assignment** -> Use UNION to combine data from two subsets in the employees_10 database. The first subset should contain the employee number (emp_no), first name (first_name), and last name (last_name) of all employees whose family name is 'Bamford'. The second subset should contain the department number (dept_no) and start date (from_date) of all managers, as recorded in the departments manager table (dept_manager). Ensure to provide null values in all empty columns for each subset. --

# Solution
SELECT 
    e.emp_no,
	e.first_name,
	e.last_name,
	NULL AS dept_no,
	NULL AS from_date
FROM
    employees e
WHERE
    last_name = 'Bamford' UNION SELECT 
    NULL AS emp_no,
	NULL AS first_name,
	NULL AS last_name,
	dm.dept_no,
	dm.from_date
FROM
    dept_manager dm;


-- **Stored routines**--
-- **Assignment** -> Create a procedure that will provide the average salary of all employees. --

# Solution
DELIMITER $$
CREATE PROCEDURE avg_salary()
BEGIN
                SELECT
                                AVG(salary)
                FROM
                                salaries;
END$$
DELIMITER ;
CALL avg_salary;
CALL avg_salary();
CALL employees.avg_salary;
CALL employees.avg_salary();


-- **Window functions**--
-- **Assignment** -> Create a query that upon execution returns a result set containing the employee numbers, contract salary values, start, and end dates of the first ever contracts that each employee signed for the company.--

# Solution
SELECT
    s1.emp_no, s.salary, s.from_date, s.to_date
FROM
    salaries s
        JOIN
    (SELECT
        emp_no, MIN(from_date) AS from_date
    FROM
        salaries
    GROUP BY emp_no) s1 ON s.emp_no = s1.emp_no
WHERE
    s.from_date = s1.from_date;


--**CTEs**--
-- **Assignment** -> Considering the salary contracts signed by female employees in the company, how many have been signed for a value below the average? Store the output in a column named no_f_salaries_below_avg. In a second column named total_no_of_salary_contracts, provide the total number of contracts signed by all employees in the company.
Use the salary column from the salaries table and the gender column from the employees table. Match the two tables on the employee number column (emp_no). --

# Solution
WITH cte AS (
    SELECT AVG(salary) AS avg_salary FROM salaries
),
cte2 AS (
    SELECT COUNT(salary) AS total_no_of_salary_contracts FROM salaries
)
SELECT
    SUM(CASE WHEN s.salary < c.avg_salary THEN 1 ELSE 0 END) AS no_f_salaries_below_avg,
    (SELECT total_no_of_salary_contracts FROM cte2) AS total_no_of_salary_contracts
FROM salaries s
JOIN employees e ON s.emp_no = e.emp_no AND e.gender = 'F'
JOIN cte c;


-- **Temporary Tables** --
-- **Assignment** -> Execute the following two queries.
1. Create a temporary table called salaries_adjusted_for_inflation based on the data in the salaries table. It should contain the following five fields for all employees:
   1.1   Employee number (emp_no)
   1.2  Salary value (salary)
   1.3  A field named inflation_adjusted_salary containing the salary value (salary) rounded to the nearest cent, which should be:
              - Multiplied by 6.5 if the contract start date (from_date) was between January 1, 1970, and December 31, 1989, inclusive;
              - Multiplied by 2.8 if the contract start date (from_date) was between January 1, 1990, and December 31, 1999, inclusive;
              - Multiplied by 3 for the rest of the contracts.
   1.4   The contract start date (from_date)
   1.5   The contract end date (to_date).
2. Select all the data from the temporary table just created. --

# Solution
CREATE TEMPORARY TABLE salaries_adjusted_for_inflation AS
SELECT  emp_no,
        salary,
        CASE
            WHEN from_date BETWEEN '1970-01-01' AND '1989-12-31' THEN ROUND(salary * 6.5, 2)
            WHEN from_date BETWEEN '1990-01-01' AND '1999-12-31' THEN ROUND(salary * 2.8, 2)
            ELSE ROUND(salary * 3, 2)
        END AS inflation_adjusted_salary,
        from_date,
        to_date
    FROM 
        salaries;        
SELECT * FROM salaries_adjusted_for_inflation;







