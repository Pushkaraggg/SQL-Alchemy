# SQL-Alchemy
Dive into advanced SQL concepts and techniques with this repository. Explore complex queries, performance tuning, and data analysis through practical examples and projects. Ideal for those looking to expand their SQL expertise.

#### Compute all pairs of politicians (pname1, pname2) such that pname1 invests in some company, and pname? invests in a subsidiary of that company. Sort your results by pname1, breaking ties oy pname2.For the given instance, you should return (Don, Hil) because Don invests in C1. Hil invests in C3, and C3 is a subsidiary of C1. You should not return (Don, Ron), who both invest in C1, because we don't consider a company to be a subsidlary of itself
```sql
CREATE TABLE Politician (
    pname VARCHAR(10) PRIMARY KEY,
    party CHAR(1)
);

CREATE TABLE Company (
    cname VARCHAR(10) PRIMARY KEY,
    revenue INT
);

CREATE TABLE Invested (
    pname VARCHAR(10),
    cname VARCHAR(10)
);

CREATE TABLE Subsidiary (
    parent VARCHAR(10),
    child VARCHAR(10)
);
```
```sql
-- Politicians
INSERT INTO Politician (pname, party) VALUES
('Don', 'R'),
('Ron', 'R'),
('Hil', 'D'),
('Bill', 'D');

-- Companies
INSERT INTO Company (cname, revenue) VALUES
('C1', 110),
('C2', 30),
('C3', 50),
('C4', 250),
('C5', 75),
('C6', 15);

-- Investments
INSERT INTO Invested (pname, cname) VALUES
('Don', 'C1'),
('Don', 'C4'),
('Ron', 'C1'),
('Hil', 'C3');

-- Subsidiary relationships
INSERT INTO Subsidiary (parent, child) VALUES
('C1', 'C2'),
('C2', 'C3'),
('C2', 'C5'),
('C4', 'C6');
```

```sql
CREATE VIEW subsidiary_subchilds AS
SELECT s.parent,s.child,sb.child AS subchild
FROM Subsidiary AS s
JOIN Subsidiary AS sb
ON s.child=sb.parent

CREATE VIEW invest_child AS
SELECT *
FROM  Invested AS i 
LEFT JOIN Subsidiary AS s
ON i.cname=s.parent

CREATE VIEW maintabl AS
SELECT ic.cname,ic.pname,ic.parent,ic.child,subchild
FROM invest_child AS ic
LEFT JOIN subsidiary_subchilds AS s
ON ic.cname=s.parent

SELECT CONCAT_WS(' , ', mtl.pname,i.pname) AS Investors
FROM maintabl AS mtl
INNER JOIN Invested AS i
ON mtl.subchild=i.cname
```


#### write an SQL query to find when stock quantity was below 50 units for more than two consecutive days.
```sql
CREATE TABLE supplier_inventor (
  supplier_id VARCHAR,
  product_id VARCHAR,
  stock_quantity INT,
  record_date DATE
);

INSERT INTO supplier_inventor (supplier_id, product_id, stock_quantity, record_date) VALUES
('S1', 'P1', 45, '2025-04-01'),
('S1', 'P1', 40, '2025-04-02'),
('S1', 'P1', 42, '2025-04-03'),
('S1', 'P1', 60, '2025-04-04'),
('S1', 'P1', 38, '2025-04-05'),
('S1', 'P1', 44, '2025-04-06'),
('S2', 'P2', 70, '2025-04-01'),
('S2', 'P2', 49, '2025-04-02'),
('S2', 'P2', 48, '2025-04-03'),
('S2', 'P2', 47, '2025-04-04'),
('S2', 'P2', 90, '2025-04-05');
```


```sql
WITH marked AS (
  SELECT *,
    CASE 
      WHEN stock_quantity < 50 THEN 1
      ELSE 0 
    END AS is_low_stock
  FROM supplier_inventor
),
ranked AS (
  SELECT *,
    LEAD(record_date) OVER (PARTITION BY supplier_id, product_id ORDER BY record_date) AS next_date,
    LEAD(is_low_stock) OVER (PARTITION BY supplier_id, product_id ORDER BY record_date) AS next_is_low_stock
  FROM marked
)
SELECT *
FROM ranked
WHERE is_low_stock = 1
  AND next_is_low_stock = 1
  AND next_date = record_date + INTERVAL '1 day';
```




#### Find % change in amazon revenue.
```sql
CREATE VIEW cte AS 
SELECT EXTRACT(MONTH FROM created_at) AS month,
SUM(value) AS current_rev
FROM sf_transactions
GROUP BY 1
ORDER BY 1 ASC

CREATE VIEW cte2 AS SELECT *,
LAG(current_rev,1) OVER(ORDER BY month) AS prev_revenue
FROM cte1

SELECT*,
((((current_rev-prev_revenue)/CAST(prev_revenue AS FLOAT))*100)) AS percentage
FROM cte2
WHERE prev_revenue IS NOT null
```





#### Identify customers who have invested in at least two funds with opposite performance trends (one increasing and the other decreasing) over the last 6 months.
```sql
CREATE TABLE Customerss (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(50));

CREATE TABLE Funds (
    FundID INT PRIMARY KEY,
    FundName VARCHAR(50),
    Region VARCHAR(50));

CREATE TABLE Trnsction (
    TransactionID INT PRIMARY KEY,
    CustomerID INT,
    FundID INT,
    Amount DECIMAL(10, 2),
    TransactionDate DATE,
    Timestamp DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customerss(CustomerID),
    FOREIGN KEY (FundID) REFERENCES Funds(FundID));

CREATE TABLE FundPerformance (
    FundID INT,
    PerformanceDate DATE,
    PerformanceTrend VARCHAR(10), -- 'Increasing' or 'Decreasing'
    FOREIGN KEY (FundID) REFERENCES Funds(FundID));
```
```sql
-- Insert sample data
INSERT INTO Customerss VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Charlie');
INSERT INTO Funds VALUES 
(1, 'Fund A', 'North America'), 
(2, 'Fund B', 'Europe'), 
(3, 'Fund C', 'Asia'),
(4, 'Fund D', 'North America');
INSERT INTO Trnsction VALUES
(1, 1, 1, 1000, '2024-01-01', '2024-01-01 09:00:00'),
(2, 1, 2, 2000, '2024-02-01', '2024-02-01 10:00:00'),
(3, 2, 1, 1500, '2024-03-01', '2024-03-01 11:00:00'),
(4, 2, 3, 2500, '2024-04-01', '2024-04-01 12:00:00'),
(5, 3, 4, 3000, '2024-05-01', '2024-05-01 13:00:00');
INSERT INTO FundPerformance VALUES
(1, '2024-01-01', 'Increasing'),
(1, '2024-02-01', 'Decreasing'),
(2, '2024-01-01', 'Increasing'),
(2, '2024-02-01', 'Decreasing'),
(3, '2024-01-01', 'Increasing'),
(4, '2024-01-01', 'Decreasing');
```
```sql
SELECT * FROM FundPerformance
SELECT * FROM Trnsction
SELECT * FROM Funds
SELECT * FROM Customerss


CREATE VIEW fund_perf AS  SELECT c.customerid,
LEAD(customername) OVER(PARTITION BY c.customerid) AS lead_num,
EXTRACT(MONTH FROM transactiondate) AS Month, 
customername,performancetrend
FROM Trnsction AS t
INNER JOIN Customerss AS c
ON t.customerid=c.customerid
INNER JOIN FundPerformance AS f
ON t.fundid=f.fundid
WHERE performancetrend IN('Increasing','Decreasing') 

WITH  cte2 AS (SELECT *,
CASE 
WHEN lead_num=customername THEN performancetrend ELSE NULL END AS performance_status
FROM fund_perf)

SELECT customerid,performancetrend,customername FROM cte2
WHERE performance_status IS NOT NULL
```








#### Find the top 5 performing funds within each region based on their weighted average returns, accounting for the size of investments in each fund.
```sql
CREATE VIEW region_amt  AS SELECT c.customerid,customername,
amount,f.region,t.fundid,
EXTRACT(MONTH FROM transactiondate) AS Month
FROM Trnsction AS t
INNER JOIN Customerss AS c
ON t.customerid=c.customerid
INNER JOIN Funds AS f
ON t.fundid=f.fundid

WITH per_region_sum AS (SELECT fundid,region,SUM(amount) AS total_sum
FROM region_amt
GROUP BY 1,2
ORDER BY 1)

SELECT *,(ROUND((total_sum/SUM(total_sum) OVER(PARTITION BY region))*100)) AS avg_return
FROM per_region_sum
ORDER BY fundid
```






#### Find all the users who were active for 3 consecutive days or more.
```sql
CREATE TABLE sf_events
(date DATE,account_id VARCHAR(10),user_id VARCHAR(10));

INSERT INTO sf_events (date, account_id, user_id) 
VALUES('2021-01-01', 'A1', 'U1'),
('2021-01-01', 'A1', 'U2'), ('2021-01-06', 'A1', 'U3'), ('2021-01-02', 'A1', 'U1'),
('2020-12-24', 'A1', 'U2'), ('2020-12-08', 'A1', 'U1'), ('2020-12-09', 'A1', 'U1'),
('2021-01-10', 'A2', 'U4'), ('2021-01-11', 'A2', 'U4'), 
('2021-01-12', 'A2', 'U4'), ('2021-01-15', 'A2', 'U5'), ('2020-12-17', 'A2', 'U4'), 
('2020-12-25', 'A3', 'U6'), ('2020-12-25', 'A3', 'U6'), ('2020-12-25', 'A3', 'U6'), 
('2020-12-06', 'A3', 'U7'), ('2020-12-06', 'A3', 'U6'), ('2021-01-14', 'A3', 'U6'), 
('2021-02-07', 'A1', 'U1'), ('2021-02-10', 'A1', 'U2'), ('2021-02-01', 'A2', 'U4'), 
('2021-02-01', 'A2', 'U5'), ('2020-12-05', 'A1', 'U8');
```
```sql
 SELECT * FROM sf_events

WITH cte AS (SELECT date,user_id,
LEAD(date,1) OVER(PARTITION BY user_id ORDER BY date ASC) nxt_date_active
FROM sf_events),

cte2 AS (SELECT user_id,date,nxt_date_active,
LEAD(date,2) OVER(PARTITION BY user_id ORDER BY date ASC) AS nxt_nxt_date_active
FROM cte),

cte3 AS (SELECT *,CASE WHEN date=nxt_date_active-1 AND
nxt_date_active=nxt_nxt_date_active-1 THEN 1 END AS cons_num
FROM cte2)

SELECT user_id,date,nxt_date_active,nxt_nxt_date_active
FROM cte3
WHERE cons_num='1'
```





#### Find the top three distinct salaries for each department. Output the department name and the top 3 distinct salaries by each department. Order your results alphabetically by department and then by highest salary to lowest.

```sql
CREATE TABLE emploee 
(id INT PRIMARY KEY,first_name VARCHAR(50), 
last_name VARCHAR(50), age INT, sex VARCHAR(1), 
employee_title VARCHAR(50), department VARCHAR(50), 
salary INT, target INT, bonus INT, city VARCHAR(50), 
address VARCHAR(50), manager_id INT);
```
```sql
INSERT INTO emploee
(id, first_name, last_name, age, sex, employee_title, department, salary, target, bonus, city, address, manager_id) 
VALUES (1, 'Allen', 'Wang', 55, 'F', 'Manager', 'Management', 200000, 0, 300, 'California', '23St', 1), 
(13, 'Katty', 'Bond', 56, 'F', 'Manager', 'Management', 150000, 0, 300, 'Arizona', NULL, 1), 
(19, 'George', 'Joe', 50, 'M', 'Manager', 'Management', 100000, 0, 300, 'Florida', '26St', 1), 
(11, 'Richerd', 'Gear', 57, 'M', 'Manager', 'Management', 250000, 0, 300, 'Alabama', NULL, 1),
(10, 'Jennifer', 'Dion', 34, 'F', 'Sales', 'Sales', 100000, 200, 150, 'Alabama', NULL, 13), 
(18, 'Laila', 'Mark', 26, 'F', 'Sales', 'Sales', 100000, 200, 150, 'Florida', '23St', 11), 
(20, 'Sarrah', 'Bicky', 31, 'F', 'Senior Sales', 'Sales', 200000, 200, 150, 'Florida', '53St', 19),
(21, 'Suzan', 'Lee', 34, 'F', 'Sales', 'Sales', 130000, 200, 150, 'Florida', '56St', 19), 
(22, 'Mandy', 'John', 31, 'F', 'Sales', 'Sales', 130000, 200, 150, 'Florida', '45St', 19),
(17, 'Mick', 'Berry', 44, 'M', 'Senior Sales', 'Sales', 220000, 200, 150, 'Florida', NULL, 11), 
(12, 'Shandler', 'Bing', 23, 'M', 'Auditor', 'Audit', 110000, 200, 150, 'Arizona', NULL, 11), 
(14, 'Jason', 'Tom', 23, 'M', 'Auditor', 'Audit', 100000, 200, 150, 'Arizona', NULL, 11), 
(16, 'Celine', 'Anston', 27, 'F', 'Auditor', 'Audit', 100000, 200, 150, 'Colorado', NULL, 11), 
(15, 'Michale', 'Jackson', 44, 'F', 'Auditor', 'Audit', 70000, 150, 150, 'Colorado', NULL, 11),
(6, 'Molly', 'Sam', 28, 'F', 'Sales', 'Sales', 140000, 100, 150, 'Arizona', '24St', 13), 
(7, 'Nicky', 'Bat', 33, 'F', 'Sales', 'Sales', NULL, NULL, NULL, NULL, NULL, NULL);
```
```sql
SELECT * FROM emploee
ORDER BY id

WITH dept_sal AS (SELECT DISTINCT
salary,department,
DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS sal_rank
FROM emploee
WHERE salary IS NOT NULL)

SELECT 
department ,salary
FROM dept_sal
WHERE sal_rank <='3'
ORDER BY department ASC,
salary DESC

--joining employee with manager name
SELECT CONCAT_WS(' ',e1.first_name,e1.last_name) AS emp_name,e1.employee_title,
e1.department,CONCAT_WS(' ',e2.first_name,e2.last_name) AS manager_name
FROM emploee AS e1
INNER JOIN emploee AS e2
ON e1.manager_id=e2.id
```








#### Estimate the growth of Airbnb each year using the number of hosts registered as the growth metric.The rate of growth is calculated by taking ((number of hosts registered in the current year - number of hosts registered in the previous year) / the number of hosts registered in the previous year) * 100.Output the year, number of hosts in the current year, number of hosts in the previous year, and the rate of growth. Round the rate of growth to the nearest percent and order the result in the ascending order based on the year.
```sql
CREATE TABLE airbnb_search_detailsss
(id INT PRIMARY KEY, price FLOAT, property_type VARCHAR(100), 
room_type VARCHAR(100), amenities VARCHAR(100), accommodates INT, 
bathrooms INT, bed_type VARCHAR(50), cancellation_policy VARCHAR(50), 
cleaning_fee INT8, city VARCHAR(100), host_identity_verified VARCHAR(10),
host_response_rate VARCHAR(10), host_since DATE, 
neighbourhood VARCHAR(100), number_of_reviews INT,
review_scores_rating FLOAT, zipcode INT, bedrooms INT, beds INT);
```
```sql
INSERT INTO airbnb_search_detailsss
(id, price, property_type, room_type, amenities, accommodates, bathrooms, bed_type, cancellation_policy, 
cleaning_fee , city, host_identity_verified, host_response_rate, host_since, neighbourhood, number_of_reviews, 
review_scores_rating, zipcode, bedrooms, beds)
VALUES(1, 100, 'Apartment', 'Entire home/apt', 'WiFi, Kitchen', 2, 1, 'Real Bed', 'Flexible', 1, 'New York', 'Yes', '90%', '2019-01-15', 'Manhattan', 120, 4.8, 10001, 1, 1), 
(2, 75, 'House', 'Private room', 'WiFi, Parking', 3, 1, 'Queen Bed', 'Moderate', 0, 'Los Angeles', 'Yes', '80%', '2018-06-22', 'Hollywood', 80, 4.5, 90001, 2, 1), 
(3, 50, 'Shared Room', 'Shared room', 'WiFi', 1, 1, 'Single Bed', 'Strict', 0, 'Chicago', 'No', '70%', '2019-03-10', 'Lincoln Park', 40, 3.8, 60614, 1, 1), 
(4, 200, 'Villa', 'Entire home/apt', 'Pool, WiFi', 6, 3, 'King Bed', 'Flexible', 1, 'Miami', 'Yes', '95%', '2020-07-05', 'Miami Beach', 300, 4.9, 33139, 3, 4), 
(5, 120, 'Apartment', 'Entire home/apt', 'WiFi, Kitchen, Parking', 4, 2, 'Double Bed', 'Moderate', 1, 'San Francisco', 'Yes', '85%', '2021-09-18', 'Downtown', 150, 4.7, 94102, 2, 2), 
(6, 80, 'Apartment', 'Private room', 'WiFi', 2, 1, 'Queen Bed', 'Strict', 0, 'Austin', 'No', '75%', '2020-11-22', 'Downtown', 100, 4.4, 78701, 1, 1),
(7, 90, 'Apartment', 'Private room', 'WiFi', 2, 1, 'single Bed', 'Strict', 0, 'Chicago', 'No', '85%', '2019-11-22', 'Downtown', 100, 4.4, 78710, 1, 1),
(8, 70, 'Apartment', 'Private room', 'WiFi', 2, 1, 'medium Bed', 'Flexible', 0, 'Austin', 'No', '75%', '2022-10-22', 'Downtown', 100, 4.4, 78751, 2, 1),
(9, 100, 'Apartment', 'Private room', 'WiFi', 2, 1, 'Queen Bed', 'Moderate', 0, 'Los Angeles', 'No', '75%', '2018-11-22', 'Downtown', 100, 4.4, 98701, 1, 1),
(10, 80, 'Apartment', 'Private room', 'WiFi', 2, 1, 'Queen Bed', 'Strict', 0, 'Austin', 'No', '75%', '2021-11-22', 'Downtown', 100, 4.4, 68701, 1, 1)
```
```sql
SELECT * FROM airbnb_search_detailsss

CREATE VIEW detail_yearss AS SELECT *, EXTRACT(YEAR FROM host_since)AS year
FROM airbnb_search_detailsss

WITH cte2 AS (SELECT year,COUNT(DISTINCT id) AS host_num_current
FROM detail_yearss
GROUP BY 1
ORDER BY year ASC),

 cte3 AS (SELECT *,
LAG(host_num_current) OVER(ORDER BY year) AS host_num_prev
FROM cte2)

SELECT year,host_num_current,host_num_prev,
(((host_num_current-host_num_prev)/(CAST (host_num_prev AS FLOAT)))*100) AS growth_rate
FROM cte3
```







#### Calculate the share of new and existing users for each month in the table. Output the month, share of new users, and share of existing users as a ratio ,New users are defined as users who started using services in the current month (there is no usage history in previous months). Existing users are users who used services in the current month but also used services in any previous month. Assume that the dates are all from the year 2020

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    time_id TIMESTAMP,
    user_id VARCHAR(50),
    customer_id VARCHAR(50),
    client_id VARCHAR(50),
    event_type VARCHAR(50),
    event_id INT);
```
```sql
INSERT INTO events (time_id, user_id, customer_id, client_id, event_type, event_id)
VALUES
('2020-01-05 12:00:00', 'user1', 'company1', 'mobile', 'login', 101),
('2020-01-06 14:00:00', 'user2', 'company1', 'desktop', 'login', 102),
('2020-02-01 15:00:00', 'user1', 'company1', 'mobile', 'logout', 103),
('2020-02-02 16:00:00', 'user3', 'company2', 'desktop', 'login', 104),
('2020-03-01 10:00:00', 'user4', 'company3', 'mobile', 'video call', 105),
('2020-03-03 11:00:00', 'user1', 'company1', 'mobile', 'login', 106),
('2020-03-04 12:00:00', 'user5', 'company3', 'desktop', 'voice call', 107),
('2020-04-01 14:00:00', 'user2', 'company1', 'desktop', 'logout', 108),
('2020-04-02 15:00:00', 'user6', 'company4', 'mobile', 'login', 109),
('2020-05-01 17:00:00', 'user7', 'company5', 'desktop', 'login', 110),
('2020-05-01 17:00:00', 'user6', 'company4', 'mobile', 'logout', 111);
```
```sql
SELECT * FROM events

CREATE VIEW new_exis_user AS WITH cte AS (SELECT *,EXTRACT(MONTH FROM time_id) AS month
FROM events),
cte2 AS (SELECT *,
RANK() OVER(PARTITION BY user_id ORDER BY Month) AS rn
FROM cte),
cte3  AS (SELECT user_id,month,
CASE WHEN rn=1 THEN 'New user' ELSE 'Existing user' END AS user_status
FROM cte2)
SELECT month,
COUNT(CASE WHEN user_status='New user' THEN 1 END) AS new_user,
COUNT(CASE WHEN user_status='Existing user' THEN 1 END) AS existing_user
FROM cte3
GROUP BY month
ORDER BY month

SELECT month,new_user,existing_user,
(new_user/(CAST((new_user+existing_user) AS FLOAT))) AS new_user_ratio,
(existing_user/(CAST((new_user+existing_user)AS FLOAT))) AS existing_user_ratio
FROM new_exis_user
```







#### You are given the table with titles of recipes from a cookbook and their page numbers. You are asked to represent how the recipes will be distributed in the book.Produce a table consisting of three columns:left_page_number, left_title and right_title. The k-th row (counting from 0), should contain the number and the title of the page with the number 2xk in the first and second columns respectively, and the title of the page with the number 2xk+1 in the third column.
```sql
CREATE TABLE cookbook_titles 
(page_number INT PRIMARY KEY,title VARCHAR(255));

INSERT INTO cookbook_titles (page_number, title) 
VALUES (1, 'Scrambled eggs'), (2, 'Fondue'), (3, 'Sandwich'), 
(4, 'Tomato soup'), (6, 'Liver'), (11, 'Fried duck'), 
(12, 'Boiled duck'), (15, 'Baked chicken');
SELECT * FROM cookbook_titles
```
```sql
CREATE VIEW right_odd AS SELECT page_number,title            --- right page as odd no.(given in ques 2k+1)
FROM cookbook_titles
 WHERE page_number%2=1  
 
CREATE VIEW left_even AS SELECT page_number,title                        
FROM cookbook_titles
WHERE page_number%2=0                                        --- left page as even no.(given in ques 2k)


SELECT l.page_number AS left_page_num,
l.title AS left_title,
r.title AS right_title
FROM left_even AS l
LEFT JOIN right_odd AS r
ON l.page_number+1 =r.page_number
```







#### You are given a table of tennis players and their matches that they could either win (W) or lose (L). Find the longest streak of wins. A streak is a set of consecutive won matches of one player. The streak ends once a player loses their next match. Output the ID of the player or players and the length of the streak.

```sql
CREATE TABLE players_results (match_date date , match_result VARCHAR(1), player_id BIGINT);

INSERT INTO players_results (match_date, match_result, player_id) 
VALUES 
('2023-01-01', 'W', 1), ('2023-01-02', 'W', 1), ('2023-01-03', 'L', 1), 
('2023-01-04', 'W', 1), ('2023-01-01', 'L', 2), ('2023-01-02', 'W', 2), 
('2023-01-03', 'W', 2), ('2023-01-04', 'W', 2), ('2023-01-05', 'L', 2), 
('2023-01-01', 'W', 3), ('2023-01-02', 'W', 3), ('2023-01-03', 'W', 3), 
('2023-01-04', 'W', 3), ('2023-01-05', 'L', 3);
```
```sql
SELECT * FROM players_results

WITH new_date AS (SELECT 
player_id,
match_date,
match_result ,
DENSE_RANK() OVER(PARTITION BY player_id ORDER BY match_date ASC),
match_date+INTERVAL'1 day' AS date
FROM 
players_results
WHERE 
match_result='W'),

lead_dates AS (SELECT * ,
LEAD(match_date) OVER(PARTITION BY player_id) AS lead_date
FROM new_date),

final_dates AS (SELECT * 
FROM lead_dates
WHERE date=lead_date 
OR lead_date IS NULL)

SELECT player_id,
COUNT(dense_rank) AS length
FROM final_dates
GROUP BY 1
ORDER BY length DESC
LIMIT 1
```








#### Find the average absolute fare difference between a specific passenger and all passengers that belong to the same pclass, both are non-survivors and age difference between two of them is 5 or less years. Do that for each passenger (that satisfy above mentioned coniditions). Output the result along with the passenger name.

```sql
CREATE TABLE 
titanic (passengerid BIGINT PRIMARY KEY, name VARCHAR(255), pclass BIGINT, survived BIGINT, age FLOAT, 
fare FLOAT, cabin VARCHAR(50), embarked VARCHAR(1), parch BIGINT, sibsp BIGINT, ticket VARCHAR(50), sex VARCHAR(10));

INSERT INTO titanic 
(passengerid, name, pclass, survived, age, fare, cabin, embarked, parch, sibsp, ticket, sex) 
VALUES (1, 'John Smith', 1, 0, 35, 71.28, 'C85', 'C', 0, 1, 'PC 17599', 'male'), 
(2, 'Mary Johnson', 1, 0, 30, 53.1, 'C123', 'C', 0, 0, 'PC 17601', 'female'), 
(3, 'James Brown', 1, 1, 40, 50.0, NULL, 'S', 0, 0, '113803', 'male'), 
(4, 'Anna Davis', 2, 0, 28, 13.5, NULL, 'S', 0, 1, '250644', 'female'), 
(5, 'Robert Wilson', 2, 0, 32, 13.5, NULL, 'S', 0, 1, '250655', 'male'), 
(6, 'Emma Moore', 3, 0, 25, 7.25, NULL, 'S', 0, 0, '349909', 'female'), 
(7, 'William Taylor', 3, 0, 27, 7.75, NULL, 'Q', 0, 0, 'STON/O 2. 3101282', 'male'), 
(8, 'Sophia Anderson', 3, 1, 22, 8.05, NULL, 'S', 0, 0, '347082', 'female'), 
(9, 'David Thomas', 1, 0, 36, 71.28, 'C85', 'C', 0, 1, 'PC 17599', 'male'), 
(10, 'Alice Walker', 1, 0, 33, 53.1, 'C123', 'C', 0, 0, 'PC 17601', 'female');
```
```sql
WITH cte AS(
SELECT   -- for comparsion always use self joins
ABS(t1.fare-t2.fare) AS nfare,
t1.passengerid,t1.name AS fname 
,t1.fare,t1.age,
t2.passengerid,
t2.name,t2.fare,t2.age,
FROM titanic AS t1
INNER JOIN 
titanic AS t2
ON t1.pclass=t2.pclass
WHERE 
t1.passengerid!=t2.passengerid
AND (t1.age-t2.age)<=5 AND 
t1.survived=0 AND t2.survived=0)

SELECT fname,AVG(nfare) AS avg_fare
FROM cte
GROUP BY fname
```









#### You are given a table EmployeeLogs with columns EmployeeID, LoginTime, LogoutTime, and Date.Write a query to calculate the longest continuous working streak (consecutive days without missing a login) for each employee.
```sql
CREATE TABLE EmployeeLogs (
    EmployeeID INT,
    LoginTime TIME,
    LogoutTime TIME,
    Date DATE
);

-- Insert sample data into the table
INSERT INTO EmployeeLogs (EmployeeID, LoginTime, LogoutTime, Date) VALUES
    (1, '09:00:00', '17:00:00', '2024-07-01'),
    (1, '09:00:00', '17:00:00', '2024-07-02'),
    (1, '09:00:00', '17:00:00', '2024-07-03'),
    (1, '09:00:00', '17:00:00', '2024-07-05'), -- Gap on July 4th
    (1, '09:00:00', '17:00:00', '2024-07-06'),
    (1, '09:00:00', '17:00:00', '2024-07-07'),
    (2, '09:00:00', '17:00:00', '2024-07-01'),
    (2, '09:00:00', '17:00:00', '2024-07-02'),
    (2, '09:00:00', '17:00:00', '2024-07-03'),
    (2, '09:00:00', '17:00:00', '2024-07-04'),
    (2, '09:00:00', '17:00:00', '2024-07-05'),
    (2, '09:00:00', '17:00:00', '2024-07-06');
```
```sql
SELECT * FROM EmployeeLogs

WITH new_dates AS (SELECT employeeid,
date,(date + INTERVAL '1 day') AS next_date,
LEAD(date) OVER (PARTITION BY employeeid ORDER BY date) AS lead_date
FROM EmployeeLogs
),
grouped_dates AS(SELECT 
employeeid,
date,
lead_date,next_date,
-- Group dates into streaks based on consecutive days (sum will automatically break at first non consecutive and will continue it.)
SUM(CASE WHEN lead_date = next_date THEN 0 ELSE 1 END) OVER(PARTITION BY employeeid ORDER BY date)AS streak_group
FROM new_dates)

SELECT employeeid,  
COUNT(streak_group) AS days_in_streak
FROM grouped_dates
WHERE streak_group=0
GROUP BY employeeid
```








#### Find cumulative percentage of total sales for each region.

```sql
Create table
CREATE TABLE SalesByRegion (
    Region VARCHAR(50),
    Order_Date DATE,
    Order_ID INT,
    Amount INT
);

-- Insert data
INSERT INTO SalesByRegion VALUES
('North', '2024-01-15', 1, 500),
('North', '2024-03-05', 5, 600),
('South', '2024-01-20', 2, 700),
('East', '2024-02-18', 4, 200);
```
```sql
SELECT * FROM SalesByRegion

WITH cte AS(SELECT region,order_Date,order_ID,amount,SUM(amount) OVER(PARTITION BY region) AS sum,
SUM(amount) OVER(PARTITION BY region ORDER BY order_Date ASC ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS total_amount
FROM SalesByRegion)

SELECT region,(total_amount/CAST(sum AS FLOAT)*100) AS cumulative_percent
FROM cte
```








#### Find first and last orders  placed by each customers based on order_date.Also find differnce in days between first and last orders.
```sql
 Create table
CREATE TABLE CustomerOrders (
    Order_ID INT,
    Customer_ID INT,
    Order_Date DATE,
    Amount INT
);

-- Insert data
INSERT INTO CustomerOrders VALUES
(1, 101, '2024-01-15', 500),
(2, 102, '2024-01-20', 700),
(3, 101, '2024-02-10', 300),
(4, 103, '2024-02-18', 200),
(5, 101, '2024-03-05', 600)
```
```sql
SELECT * FROM CustomerOrders

WITH cte AS(SELECT order_ID,customer_id,order_date,
CASE WHEN order_date=MIN(Order_Date) OVER(PARTITION BY customer_ID) THEN order_date END AS min_date,
Amount
FROM Customerorders),

cte2 AS(SELECT order_ID,customer_id,order_date,
CASE WHEN order_date=MAX(Order_Date) OVER(PARTITION BY customer_ID) THEN order_date END AS max_date,
Amount
FROM customerorders)

SELECT cte.order_ID,cte.customer_id,min_date,max_date,
(max_date-min_date) AS days_diff,
cte.amount
FROM cte
INNER JOIN cte2 
ON cte.customer_id=cte2.customer_id
WHERE min_date IS NOT NULL
AND max_date IS NOT NULL
```








#### 27Th QUES linkedin GOLDMAN SACHS (NISHANT KUMAR)
```sql
SELECT * FROM train_departures
SELECT * FROM train_arrival

WITH train_status AS(SELECT train_id,arrival_time AS event_time,1 AS event_type 
FROM train_arrival
UNION ALL
SELECT train_id,ddeparture_time AS event_time,-1 AS event_type 
FROM train_departures),

platforms AS (SELECT train_id,
SUM(event_type) OVER(ORDER BY event_time ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS platforms_needed
FROM train_status)

SELECT MAX(platforms_needed)
FROM platforms
```







#### Write a query to find the Market Share at the Product Brand level for each Territory, for Time Period Quarter4-2021. Market Share is the number of Products of a certain Product Brand sold in a territory, divided by the total number of Products sold in this Territory.Output the ID of the Territory, name of the Product Brand and the corresponding Market Share in percentages.Only include these Product Brands that had at least one sale in a given territory.

```sql
SELECT * FROM fct_customer_sales
SELECT * FROM map_customer_territory
SELECT * FROM dim_product

WITH product_info AS (SELECT 
fc.cust_id,fc.prod_sku_id AS sku_id,order_date,
territory_id,prod_sku_name AS products,prod_brand AS brands
FROM fct_customer_sales AS fc
INNER JOIN 
map_customer_territory AS mc
ON fc.cust_id=mc.cust_id
INNER JOIN 
dim_product AS dp
ON fc.prod_sku_id=dp.prod_sku_id
WHERE 
fc.order_date 
BETWEEN '2021-10-01' AND '2021-12-31'), -- 4 Quarter

brand_info AS (SELECT 
territory_id,brands,
COUNT(products) AS prod_count
FROM product_info
GROUP BY 1,2),

prod_info AS (SELECT territory_id,
SUM(prod_count) AS total_prod
FROM brand_info
GROUP BY territory_id),

all_info AS (SELECT pi.territory_id,brands,
prod_count,total_prod
FROM brand_info AS bi
INNER JOIN prod_info AS pi
ON bi.territory_id=pi.territory_id)

SELECT territory_id,brands,
ROUND((prod_count/total_prod)*100,2) AS market_share
FROM all_Info
```


#### Retrieve Housing Data from Specific Cities
#### You want to find all Airbnb listings in San Francisco and New York that have at least 10 reviews and an average rating equal to or above 4.5.
```sql
CREATE TABLE listings 
( listing_id int8 PRIMARY KEY,
name varchar(50), 
city varchar(50), 
reviews_count int
);

CREATE TABLE reviews 
(listing_id int8 PRIMARY KEY, 
review_id int8, 
stars int, 
submit_date date
);
```
```sql
SELECT * FROM reviews
SELECT * FROM listings

WITH my_cte AS (SELECT l.listing_id,ROUND(AVG(r.stars),1) AS avg_stars
FROM listings AS l
INNER JOIN reviews AS r
ON l.listing_id=r.listing_id
WHERE city IN ('New York','San Francisco')
AND reviews_count >='10'
GROUP BY 1
)

SELECT * FROM my_cte 
WHERE avg_stars>=4.5
```


#### Find the Average Number of Guests per Booking in Each City for Airbnb
```sql
CREATE TABLE bookings
(booking_id int8,
property_id int8, 
guests int8, 
booking_date date);

CREATE TABLE properties
(property_id int8, 
city varchar(50)
);
```
```sql
SELECT * FROM bookings
SELECT * FROM properties

SELECT booking_id,
city, AVG(guests) AS avg_num_guests
FROM bookings AS b
INNER JOIN properties AS p
ON b.property_id=p.property_id
GROUP BY booking_id,city
```

#### Analyzing Click-Through Rates for Airbnb Listing Views and Bookings The CTR is calculated by dividing the number of bookings by the number of listing views, giving a proportion of views that resulted in a booking.
```sql
CREATE TABLE listing_views
(view_id int8,
user_id int8,
visit_date date,
listing_id int8 );
SELECT * FROM bookings
SELECT * FROM listing_views

CREATE TABLE listings_views AS
SELECT listing_id,
COUNT(view_id) AS total_views_per_listing
FROM listing_views 
GROUP BY 1

CREATE TABLE total_bookingss AS 
SELECT COUNT(booking_id) AS total_bookings
FROM bookings 


SELECT*, 
(total_bookingss/total_views_per_listing) AS CTR FROM 
listings_views AS l 
INNER JOIN total_bookings AS b
ON l.listing_id=b.total_bookings
```



 





















