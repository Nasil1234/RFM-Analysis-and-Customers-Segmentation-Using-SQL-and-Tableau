# RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau
RFM analysis, is a type of customer segmentation and behavioral targeting used to help businesses rank and segment customers based on the recency, frequency, and monetary value of a transaction.

**Explore the Data**
Inspect the data, we have to make sure the columns required for RFM analysis exist (last purchase date, total orders, and total money spent). In this case, we use MySQL Workbench and query.

select customer_id, gender, age, total_orders, price, payment_method, shopping_mall,
substring_index(invoice_date, ' ', 1) invoice_date
from customer_shopping_data;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/dcb7779e-9522-4cc0-972a-654847bd5689)

customer_id: unique code for each customer;
gender: male or female customers gender;
age: range number of customers age;
total_orders: shows how frequently customers order;
price: total money spent by customers;
payment_method: method of customers payment;
shopping_mall: location of customers’ purchase
invoice_date: last customer’s orders date.
Then, let’s see if the data is usable to analyze.

**-- inspect data**

select count(*) -- check how many records
from customer_shopping_data;

select count(distinct(customer_id)) -- customer id has to be unique, check if it's correct
from customer_shopping_data;

select *
from customer_shopping_data -- check if there are null values
where shopping_mall is null;
The data is as good as we need it, great! Now, we can move on to explore the data we have.

-- spending by gender

select gender, sum(total_orders) orders, sum(price) sales
from customer_shopping_data
group by gender
order by 3 desc;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/44e158c1-408f-4145-b012-835b4426fb1b)

As we speculated, women tend to shop greater than men.

-- spending by mall

select shopping_mall, sum(total_orders) orders, sum(price) sales
from customer_shopping_data
group by shopping_mall
order by 3 desc;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/e1a953fa-1407-4dae-a555-65cc1ab126e1)

Mall of Istanbul has the most sales, whilst Cehavier AVM is the least.

-- top spending by year

select distinct(year(invoice_date)) year, gender, sum(total_orders) sales, sum(price) revenue
from customer_shopping_data
group by year, gender
order by revenue desc;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/cbcf1cf7-2e65-466a-8ad9-40b66c3a311c)

In 2021 and 2022, women tend to shop best.

Recency, Frequency, and Monetary
This is where the fun begins. As we know beforehand, we already have columns that contain recency, frequency, and monetary (columns: invoice_date, total_orders, and price).

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/944121f5-b4c7-4f3b-b718-c1e60f804814)

Unless we need to manipulate the invoice_date to more usable data. We’re going to calculate the different days between the last customer's purchase date to the current date (in this case, let’s say the current date is 10 March 2023).

select
  customer_id, gender, age, payment_method, shopping_mall, 
  datediff('2023-03-10', invoice_date) last_date_order,
  sum(total_orders) total_orders,
  sum(price) spending
from customer_shopping_data
group by customer_id, gender, age, payment_method, shopping_mall, invoice_date
order by last_date_order

For further analysis, we have to make CTE for our previous query, then find the percentile of last_date_order, total_orders, and spending column.

with rfm as (
 select
  customer_id, gender, age, payment_method, shopping_mall, 
  datediff('2023-03-10', invoice_date) last_date_order,
  sum(total_orders) total_orders,
  sum(price) spending
 from customer_shopping_data
 group by customer_id, gender, age, payment_method, shopping_mall, invoice_date
 order by last_date_order
)
select *,
  ntile(3) over (order by last_date_order) rfm_recency,
  ntile(3) over (order by total_orders) rfm_frequency,
  ntile(3) over (order by spending) rfm_monetary
 from rfm

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/943c1368-b98d-4218-8000-91330bedfd49)

We just coded those columns. Now, calculate the total RFM score and code for segmentation purposes.

with rfm as (
 select
  customer_id, gender, age, payment_method, shopping_mall, 
  datediff('2023-03-10', invoice_date) last_date_order,
  sum(total_orders) total_orders,
  sum(price) spending
 from customer_shopping_data
 group by customer_id, gender, age, payment_method, shopping_mall, invoice_date
 order by last_date_order
),

rfm_calc as (
 select *,
  ntile(3) over (order by last_date_order) rfm_recency,
  ntile(3) over (order by total_orders) rfm_frequency,
  ntile(3) over (order by spending) rfm_monetary
 from rfm
 order by rfm_monetary desc
)

select *, rfm_recency + rfm_frequency + rfm_monetary as rfm_score,
concat(rfm_recency, rfm_frequency, rfm_monetary) as rfm
from rfm_calc

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/21b79a22-f9d6-4cc2-8b99-6f2afb7d40a2)

Cool. The next query is even cooler. We have the RFM code, then interpret the code into human language. We want to have these five customers segment:

New customers, for those who just purchased recently, regardless the purchase frequency and total spending;
Lost customers, for those who haven’t purchased for a long time ;
Regular customers, business as usual, very good to have them;
Loyal customers, precisely our target market, surely the customers who love our products;
Champion customers, best of the best, certainly we want to keep them forever.
-- RFM

select *, case
 when rfm in (311, 312, 311) then 'new customers'
 when rfm in (111, 121, 131, 122, 133, 113, 112, 132) then 'lost customers'
 when rfm in (212, 313, 123, 221, 211, 232) then 'regular customers'
 when rfm in (223, 222, 213, 322, 231, 321, 331) then 'loyal customers'
 when rfm in (333, 332, 323, 233) then 'champion customers'
end rfm_segment
from
(
with rfm as (
 select
  customer_id, gender, age, payment_method, shopping_mall, 
  datediff('2023-03-10', invoice_date) last_date_order,
  sum(total_orders) total_orders,
  sum(price) spending
 from customer_shopping_data
 group by customer_id, gender, age, payment_method, shopping_mall, invoice_date
 order by last_date_order
),

rfm_calc as (
 select *,
  ntile(3) over (order by last_date_order) rfm_recency,
  ntile(3) over (order by total_orders) rfm_frequency,
  ntile(3) over (order by spending) rfm_monetary
 from rfm
 order by rfm_monetary desc
)

select *, rfm_recency + rfm_frequency + rfm_monetary as rfm_score,
concat(rfm_recency, rfm_frequency, rfm_monetary) as rfm
from rfm_calc
) rfm_tb;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/6fc56fdd-fc46-4cf3-99ae-0fa5ed244e76)

And. . . we’re done here, for now. Let’s visualize the dataset so the stakeholders can spot the insight better. We’re gonna use Tableau to do the job done.

Data Visualization

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/e67a481f-29c0-46cb-9b7a-2d7860902200)

Cash is still the most popular payment method among women and men shoppers, followed by credit and debit cards.

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/d48b621d-95e7-4b17-b780-b31c5b11550a)


Mall of Istanbul has the most total orders and revenue, whilst Cevahir AVM is the least.

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/22a3a581-df1d-486c-b1ae-a0c956f6fd85)


The RFM segmentation shows our desired customer segmentation for easier targeting. It shows that the lost customers segment has 28.86% population from the dataset, with a revenue contribution of $1,651,185, which is a very large bags of money. In addition, the champion customers segment contributes the most revenue with $2,897,270. Which means? We very don’t wanna lose them. Then the new customers segment who just have a taste of our products, we have to treat them carefully because they might become our precious customers segment. And finally the in-betweens, we have to listen to them better because right now we have more priority than that.

What to Do with Them?
We have the customers segment and each of them has a customer ID. Of course in enterprise resources planning (ERP) system, contains their personal information, the very least phone number, and email. Touch them with personalized marketing and loyalty programs, with a detailed strategy:

Lost Customers
Get them back. Pull them with the best promotional we can give.
New Customers
Make their purchase frequency increase with regular promotions.
Regular and Loyal Customers
In my personal experience, we have to maintain marketing costs. I prefer not to make a further promo for these segments.
Champion Customers
The most prestige segment. Invite them if we will have stores opening, make them have a taste of our newest product launching, invite them to our town halls, etc. Make them feel great because they are.
Fun, isn’t it? Collaborate with both the operation and marketing teams to achieve great results. Note that we have to maintain our analysis because consumers’ behavior changes.

