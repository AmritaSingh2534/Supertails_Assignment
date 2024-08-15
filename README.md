use trial;
select * from [dbo].[NPS Data];
select * from [dbo].[Order Data];
select * from [dbo].[Shipment Data];

/*
As an e-commerce company we would like to gauge customer loyalty and satisfaction by asking customers a single question:
"How likely are you to recommend our products / services to a friend or colleague?".
This metric is called NPS which stands for Net Promoter Score.
Respondents answer the question on a scale from 0 (not at all likely) to 10 (extremely likely).
The customer’s order can become eligible for the NPS survey under the following conditions:
i) If the customer has not filled out any NPS survey in the last 90 days.
ii) If no other order from the customer has been eligible for the NPS survey in the last 45 days.
iii) The NPS survey form is sent on the 6th day after all shipments for the order are delivered.
     Example : If all the shipments of an order are delivered on 5th July, the order will become eligible for NPS survey on 11th July.
iv) The customer can fill out the NPS form up to the 45th day from the order date for which the survey form was sent.

Problem Statement:
	1. On a daily basis, how many customers were eligible to receive the NPS survey form in the month of June?						*/
with last_ship as (
    select o.customer_id, max(s.delivery_date) as last_date
    from [dbo].[Order Data] as o join [dbo].[Shipment Data] as s on o.order_id = s.order_id
    group by o.customer_id
),
customer_eligible as (
    select customer_id, convert(date,dateadd(day, 6, last_date)) as eligible_date
    from last_ship
    where convert(date,dateadd(day, 6, last_date)) between '2024-06-01' and '2024-06-30'
)
select ec.eligible_date, count(*) as eligible_customers
from customer_eligible as ec 
left join [dbo].[NPS Data] as n on ec.customer_id = convert(varchar, n.customer_id) and n.date >= dateadd(day, -90, ec.eligible_date)
where n.customer_id is null group by ec.eligible_date order by ec.eligible_date;

/*Problem Statement:
	2. On a daily basis, how many customers were eligible to fill the NPS survey form in the month of June?	*/
with last_ship as (
    select o.customer_id, max(s.delivery_date) as last_date
    from [dbo].[Order Data] as o join [dbo].[Shipment Data] as s on o.order_id = s.order_id
    group by o.customer_id
),
customer_eligible as (
    select customer_id, convert(date,dateadd(day, 6, last_date)) as eligible_date
    from last_ship
    where convert(date,dateadd(day, 6, last_date)) between '2024-06-01' and '2024-06-30'
),
nps_data as (
	select  date as nps_date, customer_id
	from [dbo].[NPS Data]
)
select ec.eligible_date, count(distinct ec.customer_id) as eligible_customers
from customer_eligible as ec
left join  nps_data as n on ec.customer_id = convert(varchar, n.customer_id) 
and n.nps_date >= dateadd(day, -90, ec.eligible_date) and n.nps_date < ec.eligible_date
where n.customer_id is null group by ec.eligible_date order by ec.eligible_date;
