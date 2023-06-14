# oracle-ii-shared
Shared about oracle term 2 in my course.

SELECT *
FROM employees ;

CREATE TABLE locs AS
SELECT * 
FROM locations;

SELECT *
FROM locs;

SELECT *
FROM countries;

SELECT *
FROM regions;

SELECT location_id, street_address, country_id, c.country_name, r.region_name
FROM locs
JOIN countries c
USING(country_id)
JOIN regions r
USING(region_id)
WHERE region_name = 'Europe';

SELECT *
FROM departments
WHERE location_id = (
    SELECT location_id
    FROM locs
    JOIN countries c
    USING(country_id)
    JOIN regions r
    USING(region_id)
    WHERE region_name = 'Europe'
);



SELECT *
FROM departments NATURAL JOIN (
    SELECT location_id
    FROM locs
    JOIN countries c
    USING(country_id)
    JOIN regions r
    USING(region_id)
    WHERE region_name = 'Europe' -- inline view
);

SELECT *
FROM departments 
NATURAL JOIN locs_europe;

CREATE VIEW locs_europe AS
    SELECT location_id
    FROM locs
    JOIN countries c
    USING(country_id)
    JOIN regions r
    USING(region_id)
    WHERE region_name = 'Europe';
    
SELECT * 
FROM locs_europe;


SELECT * 
FROM locs
WHERE country_id
IN(
    SELECT country_id
    FROM countries 
    JOIN regions
    USING(region_id)
    WHERE region_name = 'Europe'
);

--
INSERT INTO (SELECT * 
        FROM locs
        WHERE country_id
        IN(
            SELECT country_id
            FROM countries 
            JOIN regions
            USING(region_id)
            WHERE region_name = 'Europe'
        ) WITH CHECK OPTION
) VALUES(2391, 'St 271, Dubai Cambodia.', 'DCT', 'PP', 'PP', 'DE');

-- Will be insert into table locs
SELECT *
FROM locs;

-- Specifyin explicit default values  in the INSERT and UPDATE

CREATE TABLE test_imp_def (
    id NUMBER(5),
    salary NUMBER(7,2) DEFAULT 1000
);

INSERT INTO test_imp_def VALUES(1,2000);
INSERT INTO test_imp_def(id) VALUES(2); -- Implicit Default values
INSERT INTO test_imp_def VALUES(3,DEFAULT); -- -- Explicit Default values

UPDATE test_imp_def
SET salary = DEFAULT;

SELECT * FROM test_imp_def;

-- COPY Rows from another table
INSERT INTO test_imp_def(id, salary)
    SELECT employee_id, salary
    FROM employees
    WHERE salary > 10000;
    
--
-- Using Multi-Table insert
--
INSERT ALL
    INTO sal_history VALUES(eid, hd, sal)
    INTO mgr_history VALUES(eid, mgr_id, sal)
SELECT employee_id eid, hire_date hd, salary sal, manager_id mgr_id
FROM employees
WHERE employee_id > 200;



---- CREATE Some tables
CREATE TABLE sal_history (
    empid number(5),
    hiredate DATE,
    sal number(7,2)
);

SELECT *
FROM sal_history;

TRUNCATE sal_history;

CREATE TABLE mgr_history (
    empid number(5),
    mgr_id number(5),
    sal number(7,2)
);

SELECT *
FROM mgr_history;

TRUNCATE mgr_history;

CREATE TABLE emp_sal (
    emp_id number(5),
    commission NUMBER(2,2),
    salary number(7,2)
);
SELECT *
FROM emp_sal;

CREATE TABLE emp_history (
    emp_id number(5),
    hire_date DATE,
    salary number(7,2)
);

SELECT * 
FROM emp_history;


---- end create some table.




--
-- Conditional Insert (Insert with when then)
-- 

-- ALL
INSERT ALL 
    WHEN hire_date < '01-JAN-98' THEN
        INTO emp_history VALUES(employee_id, hire_date, salary)
    WHEN commission_pct IS NOT NULL THEN
        INTO emp_sal VALUES(employee_id, commission_pct, salary)
SELECT employee_id, hire_date, salary, commission_pct
FROM employees;
--
-- First
INSERT FIRST 
    WHEN hire_date < '01-JAN-98' THEN
        INTO emp_history VALUES(employee_id, hire_date, salary)
    WHEN commission_pct IS NOT NULL THEN
        INTO emp_sal VALUES(employee_id, commission_pct, salary)
SELECT employee_id, hire_date, salary, commission_pct
FROM employees;


SELECT *
FROM emp_history;

SELECT *
FROM emp_sal;


--
-- Pivoting Insert (Just like converted store data as column to row)
-- Convert the set of sale records from the non-relational db to relational format.

SELECT *
FROM sales_source_date;

CREATE TABLE sales_source_date (
    emp_id NUMBER(5),
    week number(1),
    mon NUMBER(7),
    tue NUMBER(7),
    wed NUMBER(7),
    thu NUMBER(7),
    fri NUMBER(7)
);

CREATE TABLE sales_info (
    emp_id NUMBER(5),
    week number(1),
    day_id number(1),
    sales number(5)
);
drop table sales_info;
SELECT * FROM sales_source_date;
INSERT INTO sales_source_date VALUES(176, 6, 3000, 4000, 5000,6000, 7000);
INSERT INTO sales_source_date(emp_id, week, mon, tue, wed, thu) VALUES(177, 6, 3200, 4200, 5200,6200);


--- convert to relational format
INSERT ALL
    WHEN mon  IS NOT NULL THEN
        INTO sales_info VALUES(emp_id, week, 1, mon)
    WHEN tue IS NOT NULL THEN
        INTO sales_info VALUES(emp_id, week, 2, tue)
    WHEN wed IS NOT NULL THEN
        INTO sales_info VALUES(emp_id, week, 3, wed)
    WHEN thu IS NOT NULL THEN
        INTO sales_info VALUES(emp_id, week, 4, thu)
    WHEN fri IS NOT NULL THEN
        INTO sales_info VALUES(emp_id, week, 5, fri)
SELECT emp_id, week, mon, tue, wed, thu, fri
FROM sales_source_date;

SELECT * 
FROM sales_info
ORDER BY 1, 3;
