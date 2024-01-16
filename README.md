# RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau
RFM analysis, is a type of customer segmentation and behavioral targeting used to help businesses rank and segment customers based on the recency, frequency, and monetary value of a transaction.

**Explore the Data**
Inspect the data, we have to make sure the columns required for RFM analysis exist (last purchase date, total orders, and total money spent). In this case, we use MySQL Workbench and query.

select customer_id, gender, age, total_orders, price, payment_method, shopping_mall,
substring_index(invoice_date, ' ', 1) invoice_date
from customer_shopping_data;

![image](https://github.com/Nasil1234/RFM-Analysis-and-Customers-Segmentation-Using-SQL-and-Tableau/assets/122611712/dcb7779e-9522-4cc0-972a-654847bd5689)
