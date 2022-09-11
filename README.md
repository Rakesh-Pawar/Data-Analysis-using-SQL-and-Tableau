# :sparkles: Data Analysis using SQL and Tableau
### Goal:
- To help users project their data by offering a huge variety of data visualization tool to choose from.

![image_sql+tableau 2](https://user-images.githubusercontent.com/112051343/189176123-0571c361-9737-43f2-9756-e96a6b7ee84e.JPG)
__________________________________________________________
# Table of Containts:
- [Problem Structure](##Problem-Structure)
- [Entity Relationship Digram](##Entity-Relationship-Digram)
- [Business Task](##Business-Task)
- [Download Data](##Download-Employee-mode-from-365-data-science)
----------------------------------------------------
## :memo: Problem Structure
-	Receive a business task.
-	Use SQL to execute a query retrieving a relevant dataset from the database.
-	Export the newly obtained data in a CSV file to use in Tableau.
-	Create a professional and understandable visualization in Tableau
-----------------
## :pencil2: Entity Relationship Digram

![ERD](https://user-images.githubusercontent.com/112051343/189185299-b0e19a68-17da-485e-b7fb-588b84e7320e.JPG)
----------
## :dart:  Business Task

### :pushpin: Task 1: 
- Create a visualization that provides a breakdown between male and female employees working in the company each year, starting from 1990.

### SQL Query:

```SQL
SELECT 
  YEAR(d.from_date) AS calender_year,
  gender,
  COUNT(d.emp_no) AS num_of_employee
FROM 
	employees_mod.t_dept_emp d 
JOIN 
  employees_mod.t_employees e ON d.emp_no=e.emp_no
GROUP BY calender_year,gender
HAVING calender_year>= 1990;
```
### Steps:

-`JOIN` employee table `t_employees` with department table `t_dept_emp` then select three columns. first `from_date` to get year from employee working in department.
- From employee table `gender` column to compare male and female and last `count` Number of employees to show breakdown value.
- Filter Condition using `having` starting year is not less 1990.

### Answer:

![sql1](https://user-images.githubusercontent.com/112051343/189266620-3361b2db-28ad-4f62-8188-244a88d04f6f.JPG)

### Chart:

![Task 1](https://user-images.githubusercontent.com/112051343/189178931-b5934f4b-2c6a-42c9-8800-560fa4492212.JPG)

### Conclusion:
- Chart shows that breakdown between male and female employees is approximate 60:40 percentage.
----------------------------------------
### :pushpin: Task 2:
- Compare the number of male managers to the number of female managers from different departments for each year, starting from 1990.

### SQL Query:

```SQL
WITH CALENDER_YEAR_CTE AS (
	SELECT
		YEAR(hire_date) AS calender_year FROM employees_mod.t_employees
	GROUP BY calender_year
	 )    # for checking manager is active employee between m.from_date to m.to_Date from t_dep_manager table. 
    SELECT 
	d.dept_name,
	ee.gender,
	m.emp_no,
	m.from_date, m.to_date,
	c.calender_year,
	CASE 
		WHEN YEAR(m.to_date)>=calender_year AND YEAR(m.from_date)<=calender_year THEN 1 ELSE 0 
    END AS active
FROM  CALENDER_YEAR_CTE c
CROSS JOIN                               # mapping every row of calender_year to each and every row of other tables
	employees_mod.t_dept_manager m
JOIN
	employees_mod.t_departments d ON m.dept_no=d.dept_no
JOIN
	employees_mod.t_employees ee ON m.emp_no=ee.emp_no
ORDER BY m.emp_no, c.calender_year;
``` 
### Steps:
- In calender_year_cte, `hire_date` column gives the year of employee is hire, which is used to filter the conditions in next step. Use group by `calender_year` to remove duplicates year
-	Take `CORSS JOIN` for mapping every row of calender_year_cte to each and every row of `t_dept_manager`. Then join `t_departments` and `t_employee` tables
-	Now select five columns, d.dept_name, ee.gender, m.emp_no, Managers starting and ending date as m.from_date & m.to_date, c.calender_year.
-	Case when statement to check manager is actively working or not, likes starting year of manager [year(from_date)] is less than hire year and ending year [year(to_date)] is grater than hire year then 1 else 0. 

### Answer:

![sql2](https://user-images.githubusercontent.com/112051343/189266875-4ef108d7-4690-4ec2-bd7f-f305bffcfb9d.JPG)

### Chart:

![Task 2](https://user-images.githubusercontent.com/112051343/189187885-d4cde7eb-c8de-4c5b-a0dd-6b8d3e8b2a40.JPG)

### Conclusion:
-	Between 1996 and 1997 Female managers is maximum active and from 1998 number is decreases continuously.
-	For male managers graph is Change, from 1990 to 1998 number is increasing max at 1998 then its decreasing. 

----------------------------------------------
### :pushpin: Task 3:
- Compare the average salary of male versus female employee in the entire company until year 2002, and add a filter allowing you to see that per each department.

### SQL Query:

```SQL
SELECT 
	gender,
  dept_name,
  round(AVG(salary),2) AS avg_salary,
  YEAR(s.from_date) AS calender_year
FROM  employees_mod.t_salaries s
JOIN
	employees_mod.t_employees e ON s.emp_no=e.emp_no
JOIN						
	employees_mod.t_dept_emp de on de.emp_no=e.emp_no
JOIN
	employees_mod.t_departments d ON de.dept_no=d.dept_no

group by gender, de.dept_no, calender_year
having calender_year <=2002
ORDER BY gender,de.dept_no;
```

### Steps:
-	In this question four columns are required, first join all required tables. Then take `gender` from `t_employee`, `depr_name` from `t_department` then `avg(salary)` and `from_date` from `t_salaries`.
-	Filter the condition, that year is less than and equals to 2002 using having statement.

### Answer:

![sql3](https://user-images.githubusercontent.com/112051343/189266881-2642b572-2005-4e60-a226-86e44a5ec474.JPG)


### Chart:

![Task 3](https://user-images.githubusercontent.com/112051343/189187951-1211467d-06ef-4c34-b31b-7f41bc23b1f1.JPG)

### Conclusion:
-	From year 1990 to 1993, average salary of male employees and female employees are same. 
- Then average salary of male is slightly more than female employees.
-----------------------------
### :pushpin: Task 4:
- Create an SQL store procedure that will allow you to obtain the average Male and Female salary per department within a certain salary range. Let this range be defined by two values the user can insert when calling the procedure.

### SQL Query:

```SQL
DROP PROCEDURE IF EXISTS filter_salary;
DELIMITER $$
CREATE PROCEDURE filter_salary (IN p_min_salary FLOAT, IN p_max_salary FLOAT)
BEGIN 
SELECT 	
	gender, d.dept_name, avg(salary) as avg_salary
FROM
	 employees_mod.t_salaries s
JOIN
	employees_mod.t_employees e ON s.emp_no=e.emp_no
JOIN						
	employees_mod.t_dept_emp de on de.emp_no=e.emp_no
JOIN
	employees_mod.t_departments d ON de.dept_no=d.dept_no
WHERE salary BETWEEN p_min_salary AND p_max_salary
GROUP BY d.dept_no,gender;
END$$
DELIMITER ;

CALL filter_salary(50000,90000);
```
### Answer:

![sql4](https://user-images.githubusercontent.com/112051343/189266896-5613305f-f612-4cc7-964a-364c9c902689.JPG)

### Chart:

![Task 4](https://user-images.githubusercontent.com/112051343/189187999-d8bbc313-83a0-404d-bbea-e2de6dcce967.JPG)

### Conclusion:
- Average salary in sales department maximun and minimum in HR department.
- Average Salary of finance, Marketing and sales department is more than the average salary of all departments.
---------------------------------
## Download Employee_mode dataset from 365 data science [here](https://github.com/UsmanNiazi/365datascience/blob/master/10.%20SQL%20%2B%20Tablue/Excercise%20Files/employees_mod.sql) 


