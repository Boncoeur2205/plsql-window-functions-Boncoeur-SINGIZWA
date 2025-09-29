# plsql-window-functions-Boncoeur-SINGIZWA


PL/SQL Window Functions Project – Shopa

Course: INSY 8311 – Database Development with PL/SQL
Instructor: Eric MANIRAGUHA
Student: Boncoeur SINGIZWA
 

Business Context

Business Name: Shopa

Shopa is an emerging e-commerce marketplace that will enable the relationship between independent sellers and customers in different parts of Africa. Shopa takes care of the logistics, customer support, and payment processing, and customers are able to buy a great variety of merchandise, including electronics, fashion, and home goods.


Data Challenge

Although Shopa has comprehensive sales and customer data,
the analytics team does not have insights into which products and sellers do well in each area,
how customer purchases evolve every month,
and how they can cluster customers based on spending behaviour.


Expected Outcome

The objective is to use the sales data of Shopa to:

⦁	Determine the best selling products and sellers by the region.
⦁	Monitor the variation of the customer expenditure over time.
⦁	Segment the customers in terms of their spending.
⦁	Assist the strategic decision-making of marketing, logistics, and seller management.


Success Criteria


1.	Best 5 Products in each region in a quarter.

Function: RANK()
Use Case: Determine what products do best in each region, based on sales revenue, each quarter.

2. Monthly Sales Totals per Customer.

Function: SUM(amount) OVER(PARTITION BY customer id ORDER by sale date)
Use Case: Monitor the total amount of money each customer has spent over time.

3. Growth of month-over-month Sales per Customer.

Function: LAG(amount) or LEAD(amount)
Use Case: Compare this month sales with the last month to see how it has grown, or gone down.

4. Quartile Customer Spending.

Function: NTILE(4)
Use Case: Segregate customers into 4 quartiles on total spending, to market to them.

5. 3-month Moving average of product sales.

Formula: AVG(amount) OVER(ROWS between 2 PRECEDING and CURRENT row)
Use Case: Flatten the peaks and niches and show the sales pattern of each product.


Database Schema

1.	customers

customer_id-INT (Primary Key)
name-VARCHAR
region-VARCHAR

'''sql

CREATE TABLE customers (customer_id INT PRIMARY KEY, name VARCHAR(100), region VARCHAR(50));

INSERT INTO customers (customer_id, name, region) VALUES
(1001, 'Amina Niyonsaba', 'Kigali'),
(1002, 'John Mugisha', 'Butare'),
(1003, 'Esther Uwizeye', 'Musanze'),
(1004, 'Eric Habimana', 'Kigali'),
(1005, 'Grace Uwimana', 'Rubavu');
'''


2. products

product_id-INT (Primary Key)
name-VARCHAR
category-VARCHAR

CREATE TABLE products (product_id INT PRIMARY KEY, name VARCHAR(100), category VARCHAR(50));

INSERT INTO products (product_id, name, category) VALUES
(2001, 'Smartphone X', 'Electronics'),
(2002, 'Bluetooth Speaker', 'Electronics'),
(2003, 'Cooking Pot Set', 'Home & Kitchen'),
(2004, 'Office Chair', 'Furniture'),
(2005, 'Backpack', 'Fashion');


3. transactions

transaction_id-INT (Primary Key)
customer_id-INT (Foreign Key)
product_id-INT (Foreign Key)
sale_date-DATE
amount-NUMERIC

CREATE TABLE transactions (transaction_id INT PRIMARY KEY, customer_id INT REFERENCES customers(customer_id), product_id INT REFERENCES products(product_id), sale_date DATE, amount NUMERIC(12, 2));

INSERT INTO transactions (transaction_id, customer_id, product_id, sale_date, amount) VALUES
(3001, 1001, 2001, '2024-01-15', 250000),
(3002, 1002, 2003, '2024-01-18', 85000),
(3003, 1003, 2002, '2024-02-10', 120000),
(3004, 1001, 2005, '2024-02-25', 35000),
(3005, 1004, 2004, '2024-03-05', 180000),
(3006, 1005, 2001, '2024-03-20', 260000),
(3007, 1002, 2002, '2024-04-01', 125000),
(3008, 1003, 2005, '2024-04-18', 40000),
(3009, 1004, 2003, '2024-05-05', 95000),
(3010, 1001, 2001, '2024-05-15', 255000),
(3011, 1005, 2004, '2024-06-01', 175000),
(3012, 1002, 2002, '2024-06-15', 130000);


Relationships

transactions.customer_id → customers.customer_id
transactions.product_id → products.product_id


SQL Window Functions


1.RANKING_QUERY

Query: select customer_id, sum(amount) as total_spent,
rank() over(order by sum(amount) desc) as spending_rank from transactions
group by customer_id
order by spending_rank;



Interpretation: This query ranks the customers according to their overall amount of spending in all purchases. It assists to pinpoint the most valuable customers of Shopa to enable the business to target the best spenders in terms of loyalty programs, customized offers, or VIP services. The equal spenders are ranked equally.

2. AGGREGATE_QUERY

Query: select customer_id, sale_date, amount,
sum(amount) over(partition by customer_id order by sale_date rows between unbounded preceding and current row) as running_total
from transactions
order by customer_id, sale_date;

Interpretation: This query is a cumulative measure of the number of dollars that each customer has spent in the past. It exposes expenditure habits and trends by individual customer, which is helpful in comprehending loyalty and life-time value, as well as buying trends. It can assist in initiating automatic rewarding on the achievement of spending milestones.

3. NAVIGATION_QUERY

Query: select customer_id, sale_date, amount,
lag(amount) over(partition by customer_id order by sale_date) as previous_amount,
amount - lag(amount) over(partition by customer_id order by sale_date) as difference
from transactions
order by customer_id, sale_date;

Interpretation: This query indicates the quantity of purchase per customer by comparing the purchase made by the customer in the transactions to the one made by the customer just before the purchase. It assists the analytics team to notice when spending drops or rises abruptly, which could be an indication of customer churn, customer satisfaction, or effective promotions. The negative differences indicate the deteriorating engagement.

4. DISTRIBUTION_QUERY

Query: select customer_id, sum(amount) as total_spent,
ntile(4) over(order by sum(amount) desc) as spending_quartile
from transactions
group by customer_id
order by spending_quartile;

Interpretation: This query puts each customer in four quartiles on the basis of the total expenditure. The percentage of customers on Quartile 1 is the 25% of the top customers and Quartile 4 covers the lowest spenders. This segmentation will enable Shopa to develop differentiated marketing, including those of exclusive offers and up-sell offers, respectively.

5. MOVING_AVERAGE_QUERY

Query: select product_id, sale_date, amount,
avg(amount) over(
partition by product_id
order by sale_date
rows between 2 preceding and current row
) as moving_avg_3_sales
from transactions
order by product_id, sale_date;

Interpretation: This query averages the short-term variation in sales by finding an average of the movement of sales of every product over the past three transactions. It assists in determining the long term sales trends and may be applied to determine the momentum of newly introduced or advertised products. Sudden shifts of moving averages can be due to seasonality or influence of marketing campaigns.

Results Analysis

1.	Descriptive: What happened?
⦁	Product sales peaked in Q2, especially for electronics.
⦁	Customers from Kigali spent more consistently than other regions.

2. Diagnostic: Why did it happen?
⦁	Promotional campaigns in April and May drove up electronics sales.
⦁	Kigali customers may benefit from faster delivery and better service quality.

3. Prescriptive: What should Shopa do?
⦁	Expand electronics promotions to Musanze and Butare.
⦁	Consider launching loyalty incentives for consistent high spenders in Kigali.


References

1. Oracle Documentation: [https://docs.oracle.com](https://docs.oracle.com)
2. Mode Analytics SQL Window Functions Guide
3. PostgreSQL Official Docs
4. The Coding Train. (2022). 'SQL Window Functions-Explained with Examples' YouTube (https://www.youtube.com/watch?v=yVnGpG2dYcM)
5. W3Schools SQL OVER() Clause
6. freeCodeCamp.org. (2023). 'SQL Tutorial - Full Course on Window Functions' YouTube (https://www.youtube.com/watch?v=7LbH-xDUdTg)
7. DataCamp – Window Functions in SQL
8. BISENGIMANA, Ivan. 'Database Management Systems Class Notes' (Lecture notes). Adventist University of Central Africa.
9. draw.io – ER Diagram tool
10. Eric, MANIRAGUHA Lecture Notes from INSY 8311


Academic Integrity Statement

All sources were properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation. This project upholds AUCA’s integrity guidelines.
