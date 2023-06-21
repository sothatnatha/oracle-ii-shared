# oracle-ii-shared
# Lession4 Manipulating Large Data Sets

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


-----------------------------------------------
-- Lesson 4 | Using Merg Statement

CREATE TABLE tbl_merg_emp (
    id number(5) PRIMARY KEY,
    name varchar2(25),
    salary number(7,2)
);

SELECT *
FROM tbl_merg_emp;

INSERT INTO tbl_merg_emp -- paste
    SELECT employee_id, last_name, salary
    FROM employees
    WHERE commission_pct IS NOT NULL; -- copy
    

MERGE INTO tbl_merg_emp des
USING(SELECT 179 AS id, 'Sothatna' AS name, 1900 AS salary FROM DUAL) src
ON (des.id=src.id)
WHEN MATCHED THEN
    UPDATE SET des.name = src.name, des.salary = src.salary
WHEN NOT MATCHED THEN
    INSERT VALUES(src.id, src.name, src.salary);
    
--
MERGE INTO tbl_merg_emp des
USING(SELECT 179 AS id, 'Sothatna' AS name, 1900 AS salary , 'Delete' AS opt_del FROM DUAL) src
ON (des.id=src.id)
WHEN MATCHED THEN
    UPDATE SET des.name = src.name, des.salary = src.salary
    DELETE WHERE (opt_del='Delete')
WHEN NOT MATCHED THEN
    INSERT VALUES(src.id, src.name, src.salary);
    
    

-- Adding column to table tbl_merg_emp
ALTER TABLE tbl_merg_emp
ADD department_id NUMBER(5);


MERGE INTO tbl_merg_emp des
USING(SELECT employee_id, last_name, salary, department_id
    FROM employees) src
ON(src.employee_id = des.id)
WHEN MATCHED THEN 
    UPDATE SET des.department_id = src.department_id
WHEN NOT MATCHED THEN 
    INSERT (des.id, des.name, des.salary, des.department_id)
    VALUES (src.employee_id, src.last_name, src.salary, src.department_id);


-- Retrieve data
SELECT *
FROM tbl_merg_emp;


-- Save data
COMMIT;

--
-- Using tracking data chages over a period time.
-- scn = system change number

UPDATE tbl_merg_emp
SET salary=2500
WHERE id = 179;

UPDATE tbl_merg_emp
SET salary=5000
WHERE id = 179;

COMMIT;

SELECT salary
FROM tbl_merg_emp
VERSIONS BETWEEN SCN MINVALUE AND MAXVALUE
WHERE id = 179;


SELECT versions_starttime, versions_endtime, salary
FROM tbl_merg_emp
VERSIONS BETWEEN SCN MINVALUE AND MAXVALUE
WHERE id = 179;

--------------------------------------------------------------------------
-- Lesson 5 | Managing Data in Difference timezone
 
CREATE TABLE tb_products(
    pid number(5) primary key,
    pname varchar2(25) UNIQUE,
    qty NUMBER(5),
    price number(7,2),
    created_date date,
    expiry_ym interval year to month,
    expiry_ds interval day to second
);

INSERT INTO tb_products
VALUES (123, 'Coke', 3000, 12, sysdate, '2-3', '15 5:30:00');

INSERT INTO tb_products
VALUES (321, 'Pepsi', 8000, 19, sysdate, INTERVAL '5' YEAR, INTERVAL '15 5' DAY TO HOUR);

SELECT pname, qty, price, created_date, expiry_ym, expiry_ds, to_char(created_date+expiry_ym+expiry_ds, 'DD-MM-YY HH:MM:SS')
FROM tb_products;

-- systdate + 2y + 3m + 15d + 15h + 30mn
-- day (+, -, *, /)
-- month_between, add_month

SELECT sysdate, to_char(add_months(sysdate, 2 * 12 + 3)+15+15/24+30/60/24, 'DD-MM-YY HH:MI:SS') new_date
FROM dual;


-- Extract Date
-- extract(day from sysdate)
-- extract(month from sysdate)
-- extract(year from sysdate)

SELECT sysdate, extract(year from sysdate)
FROM dual;



-- TZ_OFFSET

SELECT *
FROM V$TIMEZONE_NAMES;

SELECT TZ_OFFSET('Africa/Addis_Ababa')
FROM dual;

-- FROM_TZ
-- SELECT FROM_TZ(timestamp, '2000-07-12 08:00:00', 'Africa/Addis_Ababa')
-- FROM dual;

-- To_Timestamp
SELECT TO_TIMESTAMP('22-06-2023', 'dd-mm-yyyy')
FROM dual;

-- 
SELECT TO_DATE('22-06-2023', 'dd-mm-yyyy')+3
FROM dual;

-- TO_YMINTERVAL
SELECT TO_YMINTERVAL('5-3') + sysdate, TO_DSINTERVAL('10 5:5:5')
FROM dual;
