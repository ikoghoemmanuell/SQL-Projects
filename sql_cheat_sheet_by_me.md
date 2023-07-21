```sql
CREATE TABLE student(
    student_id INT AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    major VARCHAR(20) UNIQUE,
    maYYYr VARCHAR(20) DEFAULT 'undecided',
    PRIMARY KEY(student_id)
);
DROP TABLE student;

DESCRIBE student;

ALTER TABLE student ADD gpa DECIMAL(3,2);                   --EDIT COLUMNS
ALTER TABLE student DROP COLUMN gpa;

INSERT INTO student(name, major) VALUES('Jack', 'Biology'); --NEW ENTRIES
INSERT INTO student(name, major) VALUES('Kate', 'Sociology');


UPDATE student                                              --EDIT CELLS
SET major = 'Bio-Sociology'
WHERE major = 'Bio' OR major = 'Sociology';

DELETE FROM student                                         --DELETE CELLS
WHERE name = 'Tom' AND major = 'undecided';

SELECT *
FROM student                                                  --LIMIT 5
LIMIT 2;

SELECT distinct student.name, student.major                   --exclude duplicates
FROM student
ORDER BY name, major ASC/DESC;                                  --SORT

SELECT *
FROM student
WHERE student_id < 3 AND name <> 'Jack';
-- WHERE major = 'Bio' OR major = 'Sociology'               --FILTER


                                                      -- HOW TO USE HAVING
or if you used group by, you can say having.....
group by name
having...
having aggregates the rows so u can use aggregate fxns like sum, count, etc,
while where loooks at each column one by one          -- HAVING VS WHERE

SELECT *
FROM student
WHERE name IN ('Jack', 'Mike', 'Kate') AND student_id > 1;


CREATE TABLE branch (                                       --CREATE TABLE
  branch_id INT PRIMARY KEY,                                --PRIMARY KEY
  branch_name VARCHAR(40),
  mgr_id INT,
  mgr_start_date DATE,
  FOREIGN KEY(mgr_id) REFERENCES employee(emp_id) ON DELETE SET NULL
);

ALTER TABLE employee
ADD FOREIGN KEY(branch_id)                                  --FOREIGN KEY
REFERENCES branch(branch_id)
ON DELETE SET NULL;

CREATE TABLE works_with (
  emp_id INT,
  client_id INT,
  total_sales INT,
  PRIMARY KEY(emp_id, client_id),
  FOREIGN KEY(emp_id) REFERENCES employee(emp_id) ON DELETE CASCADE,
  FOREIGN KEY(client_id) REFERENCES client(client_id) ON DELETE CASCADE
);

-- Find the forename and surnames names of all employees
SELECT first_name AS forename, employee.last_name AS surname --SELECT AS
FROM employee;

-- Find all employees who are female & born after 1969 or who make over 80000
SELECT *
FROM employee
WHERE (birth_day >= '1970-01-01' AND sex = 'F') OR salary > 80000;

-- Find all employees born between 1970 and 1975
SELECT *
FROM employee
WHERE birth_day BETWEEN '1970-01-01' AND '1975-01-01';

-- Find out how many males and females there are
SELECT COUNT(sex), sex COUNT, AVG, SUM
FROM employee
GROUP BY sex;                                                   ---distinct values to CATEGORIZE

-- Find any client's who are an LLC
SELECT *
FROM client
WHERE client_name LIKE '%LLC';                                   %--ANY BEFORE



-- Find any employee born on the 10th day of the month
SELECT *
FROM employee
WHERE birth_day LIKE '_____10%';                                  --4 THINGS B4

-- Find a list of all clients & branch suppliers' names
SELECT client.client_name AS Non-Employee_Entities, client.branch_id AS Branch_ID
FROM client
UNION                                                           ---ADD TWO COLUMNS
SELECT branch_supplier.supplier_name, branch_supplier.branch_id
FROM branch_supplier;

SELECT employee.emp_id, employee.first_name, branch.branch_name
FROM employee
JOIN branch                                     -- LEFT -emp_id,    RIGHT -mgr_id
ON employee.emp_id = branch.mgr_id;


 -- Find all clients who are handles by the branch that Michael Scott manages
 -- Assume you DONT'T know Michael's ID
 SELECT client.client_id, client.client_name
 FROM client
 WHERE client.branch_id = (SELECT branch.branch_id              ---NESTED QUERY
                           FROM branch
                           WHERE branch.mgr_id = (SELECT employee.emp_id
                                                  FROM employee
                                                  WHERE employee.first_name = 'Michael' AND employee.last_name ='Scott'
                                                  LIMIT 1
                                                  ));




-----TRIGGERS-------------------------------------------------------------------

CREATE TABLE trigger_test (
     message VARCHAR(100)
);

DELIMITER $$
CREATE
    TRIGGER my_trigger BEFORE INSERT
    ON employee
    FOR EACH ROW BEGIN
         IF NEW.sex = 'M' THEN
               INSERT INTO trigger_test VALUES('added male employee');
         ELSEIF NEW.sex = 'F' THEN
               INSERT INTO trigger_test VALUES('added female');
         ELSE
               INSERT INTO trigger_test VALUES('added other employee');
         END IF;
    END$$
DELIMITER ;
INSERT INTO employee
VALUES(112, 'Pam', 'Beesly', '1988-02-19', 'M', 69000, 106, 3);


SELECT *
FROM trigger_test;

DROP TRIGGER my_trigger;


--------------------------------------------------------------------------------
SELECT DISTINCT city
FROM station
WHERE LOWER(LEFT(city, 1)) IN ('a', 'e', 'i', 'o', 'u')
  AND LOWER(RIGHT(city, 1)) not IN ('a', 'e', 'i', 'o', 'u');      -- START AND END WITH A VOWEL

-- PRINT 1 & 2 AS TWO SPACE SEPARATED INTEGERS
            SELECT CONCAT(1, ' ', 2) AS two_space_separated_integers;
            -- OR
            SELECT 1 AS first_integer, 2 AS second_integer;















```
