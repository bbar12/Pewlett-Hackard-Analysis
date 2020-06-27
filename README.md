# Pewlett-Hackard-Analysis

Given the amount of talent to be hired at PH, it is crucial to consider what talent is lost and who can provide the training necessary to the new generation of PH. The problem lies in identifying (in a large organization) the amount of employees to be retired and finding not only the gap there but finding the gap of people who can provide some level of mentorship to oncoming employees. For this, queries were performed using the following data files and using SQL to perform queries (searches) within the data in a structure we can read:
	• Departments
	• Dept_employees
	• Dept_manager
	• Employees
	• Salaries
	• Titles

To perform queries within SQL, the first step was to successfully import these files with dependencies. The files were imported and dependencies were created with the following EBD in mind:

![]EmployeeDB.png

This aid helps in identifying relationships within the separate files of data to allow for connections or joins later on when we make our queries. 
For the first deliverable, the query was initiated as selection of columns from 3 of the files: employees, titles and salaries. This is because these three files together obtain the information we seek to determine the retiring employees based on titles and salaries. These tables are connected through the use of emp_no as the key to join employees table to the titles table and is later joined through that same key to the salary table. The employees file plays a crucial role here in not only providing inner join ability but also in allowing for the joins to be finalized through the condition that the birth date range be what we specified. The final query looks like this:

SELECT e.emp_no,
	e.first_name,
	e.last_name,
	ti.title,
	s.from_date,
	s.salary
INTO retiring_employees
FROM employees as e
INNER JOIN titles as ti
ON e.emp_no = ti.emp_no
INNER JOIN salaries as s
ON e.emp_no = s.emp_no
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31');

The query is then partitioned to eliminate duplicates in the data. This also accounts for employees that have held titles in the past and only displays their most recent titles:

-- Partition the data to show only most recent title per employee
SELECT emp_no,
	first_name,
	last_name,
	title,
	from_date,
	salary
INTO new_retirees
FROM
 (SELECT emp_no,
	first_name,
	last_name,
	title,
	from_date,
	salary, ROW_NUMBER() OVER
 (PARTITION BY (emp_no)
 ORDER BY from_date DESC) rn
 FROM retiring_employees
 ) 
tmp WHERE rn = 1
ORDER BY emp_no;
SELECT * FROM new_retirees
ORDER BY emp_no;

The final tables for this are displayed in retiring_employees and in new_retirees, correspondingly. 

The second query is similar, but allows to change the birthdate range to identify the employees able to provide mentorship to oncoming talent:

--Deliverable 2: Mentorship Elegibility 
--Employee number, first and last name, title, from_date and to_date
SELECT e.emp_no,
	e.first_name,
e.last_name,
	ti.title, 
	ti.from_date,
	ti.to_date
INTO available_mentors
FROM employees as e
INNER JOIN titles as ti
ON (e.emp_no=ti.emp_no)
INNER JOIN dept_employees as de
ON (e.emp_no = de.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
--AND (e.hire_date BETWEEN '1985-01-01' AND '1988-12-31')
AND (de.to_date = '9999-01-01');

Since this also provides employees that have been with PH for a large portion of time, we need to partition the data to only include most recent titles per employee number to eliminate duplicates in the data:

--Partitioning to remove duplicates
SELECT emp_no,
	first_name,
	last_name,
	title,
	from_date,
	to_date
INTO new_mentors
FROM 
 (SELECT emp_no,
	first_name,
	last_name,
	title,
	from_date,
	to_date, ROW_NUMBER() OVER
 (PARTITION BY (emp_no)
 ORDER BY from_date DESC) rn
 FROM available_mentors
 ) 
tmp WHERE rn = 1
ORDER BY emp_no;

This data is finalized in the available_mentors and new_mentors tables, respectively. 

Some of the challenges in the queries I observed were performed in two main areas: importing tables with the right dependencies and in determining how to partition a resulting table to account for duplicates in the results. 

When creating tables, there are a number of factors that affect a successful import. Anything from a faulty dependency, to a lack of configuration in the file, to a syntax error can affect the result. Checking the tables using the SELECT format helped in seeing if I was ever in the right track before this became apparent too late into the query. For example, in my titles table creation I suffered in determining that the primary keys also needed to include from_date to be able to bridge that into the employees data

CREATE TABLE titles (
	emp_no INT NOT NULL,
	title VARCHAR(40) NOT NULL,
	from_date DATE NOT NULL,
  	to_date DATE,
	FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	PRIMARY KEY (emp_no, title, from_date)
);

Partitioning the query was a new and foreign skills as well. Determining the final syntax and trying to read it was challenging as it uses its own set of abbreviations, and it is much simpler to eliminate duplicates in other programs such as in Pandas. Having a formatted code to use and retrofit into did however help as well as the SELECT command for visualization of results. 

(PARTITION BY (emp_no)
 ORDER BY from_date DESC) rn
 FROM available_mentors
 ) 
tmp WHERE rn = 1
ORDER BY emp_no;


The results of the analysis conclude that PH currently has 499495 people available to mentor oncoming talent. This data is broad but should in the future be filtered in terms of location so that each PH site is able to have staff that can provide the training necessary. This would require having an extra data file that would account for emp_no, titles, dates of employment and locations where each employee has worked. With title changes, there may be a possibility of employees changing sites, at which point the data would have to be further partitioned to include most recent titles which would give us the most recent sites of employment. 

However, looking forward mentorship is available. The amount of retiring employees based on our data is 499996 which seems like a large number and greater than the amount of available mentors but less than 1% of a difference in staffing. This says that PH should not only be able to provide mentorship, but hopefully create more senior staff to be created as a means to continuously have fully trained talent. To this end, determining a larger amount of people that can provide mentorship might be necessary. This would require widening the data query to include a certain range of experience using from_date and to_date to determine how much experience can provide in terms of becoming senior staff and/or providing mentorship. 
