# Car_Sales_Analytics
Analytics project describing a car sales data. Please view excel charts and tables of solutions by downloading file named query1.xls <br />
Project analyst questions and solutions

## Please give an overview of sales for 2004. Breakdown by product, country, and city. Include sales value, cost of sales and net profit in 2004.


### SQL:

select t1.orderDate, t1.orderNumber, priceEach, buyPrice, productLine, productName, country, city <br />
from orders t1 <br />
inner join orderdetails t2 <br />
on t1.orderNumber = t2.orderNumber <br />
inner join products t3 <br />
on t2.productCode = t3.productCode <br />
inner join customers t4 <br />
on t1.customerNumber = t4.customerNumber <br />
where year(orderDate) = 2004 <br />


### Export to excel:

Add derived columns :- cost of sales (quantityOrdered * price each), sales value (quantityOrdered * buyPrice) and netProfit(salesValue - costOfSales) <br />
<br />
## Please give  breakdown of what products are commonly purchased together and any products rarely purchased together. <br />

Aim is to get table of rows and columns matrix showing how much product are bought together (max itemset = 2)<br />

We need to get order number as well as distinct join of products in the same order. <br />

### SQL command:
with prod_sales as <br />
( <br />
select orderNumber, t1.productCode, productLine <br />
from orderdetails as t1 <br />
inner join products as t2 <br />
on t1.productCode = t2.productCode <br />
) <br />

select distinct t1.orderNumber, t1.productLine as product1, t2.productLine as product2 <br />
from prod_sales t1 <br />
inner join prod_sales t2 <br />
on t1.orderNumber = t2.orderNumber and t1.productLine <> t2.productLine <br />

## Give breakdown of sales but also show their credit limit. Group credit limits with a high level view to see if we get higher sales for customers who have higher credit limits

### Sql: 
select t1.customerNumber, creditLimit, t3.quantityOrdered, t3.priceEach, (t3.quantityOrdered * t3.priceEach) as sales_value <br />
from customers as t1 <br />
inner join orders as t2 <br />
on t1.customerNumber = t2.customerNumber <br />
inner join orderdetails as t3 <br />
on t2.orderNumber = t3.orderNumber <br />

OR

select t1.customerNumber, creditLimit, t3.quantityOrdered, t3.priceEach <br />
from customers as t1 <br />
inner join orders as t2 <br />
on t1.customerNumber = t2.customerNumber <br />
inner join orderdetails as t3 <br />
on t2.orderNumber = t3.orderNumber <br />

OR

 Instead of pivot table in excel use sql for grouping data:<br />
from customers as t1<br />
inner join orders as t2<br />
on t1.customerNumber = t2.customerNumber<br />
inner join orderdetails as t3<br />
on t2.orderNumber = t3.orderNumber<br />
group by creditLimit<br />
order by creditLimit<br />

OR

select t2.orderNumber, t1.customerNumber, <br />
case when creditLimit < 50000 then 'a: less than £50k'<br />
when creditLimit between 50000 and 99999 then 'b: £50k - £99999'<br />
when creditLimit between 100000 and 149999 then 'c: £100k - £149999'<br />
when creditLimit > 149999 then 'd: greater than £149999'<br />
else 'other'<br />
end as creditLimitGroup, <br />
sum(t3.quantityOrdered * t3.priceEach) as sales_value<br />
from customers as t1<br />
inner join orders as t2<br />
on t1.customerNumber = t2.customerNumber<br />
inner join orderdetails as t3<br />
on t2.orderNumber = t3.orderNumber<br />
group by t2.orderNumber, t1.customerNumber, creditLimitGroup<br />


### Excel:
For non custom grouping:<br />
Calculate sales_value column if not done with sql already. <br />
Use pivot tables to group by credit limit against total sales_value if not done in sql already.<br />
Use pivot tables and graph to display creditLimit Grp against salesValue per order if orderNumber included.<br />
Plot pivot graph.<br />


## Give a view of customer sales. Include a column showing difference in value from previous sale. I want to see if new customers who make their first purchase are likely to spend more

### SQL:

with main_cte as <br />
(select customerName, t2.customerNumber, orderDate, t2.orderNumber, sum(quantityOrdered*priceEach) as order_value,<br />
row_number() over (partition by customerNumber order by orderDate) as order_date_rank<br />
from customers as t1<br />
inner join orders as t2<br />
on t1.customerNumber = t2.customerNumber <br />
inner join orderdetails as t3<br />
on t2.orderNumber = t3.orderNumber<br />
group by customerName, orderNumber, customerNumber, orderDate<br />

),<br />

second_cte as<br />
(<br />
select customerName, customerNumber, orderDate, orderNumber, order_value, <br />
lag(order_value) over (partition by customerNumber order by orderDate) as prev_order_value,<br />
order_date_rank <br />
from main_cte<br />
)<br />

select *, (order_value - prev_order_value) as order_value_difference<br />
from second_cte<br />
where prev_order_value is not null and order_date_rank<br />

### EXCEL:
Export data to excel<br />
Make pivot table of purchase_number (prev order_date_rank on sql) as row and order_value_difference as value.<br />
Plot column chart of pivot table<br />


## Show view of where customers of each office are located 

### SQL:

select t1.officeCode, t1.city as office_city, t1.country as office_country, t3.city as customer_city, t3.country as customer_country, customerNumber<br />
from offices as t1<br />
inner join employees as t2<br />
on t1.officeCode = t2.officeCode<br />
inner join customers as t3<br />
on t2.employeeNumber = t3.salesRepEmployeeNumber<br />


### Excel:

Add pivot table with office country on column and customer_country on row and add count of customer number as value.<br />
Color grade for visual ease using conditional formatting<br />


## Show sales_value by office location

### SQL:

select t1.country as office_country, t3.country as customer_country, t3.customerNumber, (t5.quantityOrdered * t5.priceEach) as sales_value<br />
from offices as t1<br />
inner join employees as t2<br />
on t1.officeCode = t2.officeCode<br />
inner join customers as t3<br />
on t2.employeeNumber = t3.salesRepEmployeeNumber<br />
inner join orders as t4<br />
on t3.customerNumber = t4.customerNumber<br />
inner join orderdetails as t5<br />
on t4.orderNumber = t5.orderNumber<br />


### Excel: 

Copy sales_value and office_country and paste into new sheet or location<br />
Create pivot table with office as row and avg sales_value as value.<br />
Plot pivot column chart<br />


## Shipping has been delayed due to the weather and may take up to 3 days to arrive. Get the list of affected orders

### SQL:

with first_cte as (select *,<br />
date_add(shippedDate, interval 3 day) as latest_arrival<br />
from orders)<br />

select *<br />
from first_cte<br />
where latest_arrival > requiredDate<br />


## Send breakdown of each customer and their sales, but include money owed column to show if customers have gone over their credit limit

### SQL:


with customer_sales as<br />
(
select t1.customerNumber, t2.orderNumber, t2.orderDate,  t3.quantityOrdered * t3.priceEach as sales_value, t1.creditLimit<br />
from customers as t1<br />
inner join orders as t2<br />
on t1.customerNumber = t2.customerNumber<br />
inner join orderdetails as t3<br />
on t2.orderNumber = t3.orderNumber<br />
),<br />

customer_payment as<br />
(<br />
select customerNumber, paymentDate, amount as amountPaid<br />
from payments<br />

),<br />

customer_order as <br />
(
select customerNumber, orderDate, <br />
lead(orderDate) over (partition by customerNumber order by orderDate) as next_order_date,<br />
 creditLimit, sum(sales_value) as orderValue<br />
from customer_sales<br />
group by customerNumber, orderDate, creditLimit<br />
),<br />

order_payment_cte as<br />
(<br />
select t1.customerNumber, orderDate, next_order_date, creditLimit, orderValue,<br />
 sum(orderValue) over (partition by t1.customerNumber order by orderDate) as running_total_order,<br />
 sum(amountPaid) over (partition by t1.customerNumber order by orderDate) as running_total_paid,<br />
 paymentDate, amountPaid<br />
from customer_order as t1<br />
left join customer_payment as t2<br />
on t1.customerNumber = t2.customerNumber and paymentDate between orderDate and case when next_order_date is null then current_date else next_order_date end<br />

)<br />

select customerNumber, creditLimit, orderDate, next_order_date,  <br />
 paymentDate,orderValue, amountPaid,running_total_order, running_total_paid, (running_total_order - running_total_paid) as money_owed,<br />
case when (running_total_order - running_total_paid) > creditLimit then (running_total_order - running_total_paid) - creditLimit <br />
else 0<br />
end as overLimitAmount<br />
from order_payment_cte<br />
order by customerNumber<br />




