---
layout: post
title:  "Hanukkah of Data - Nights 5-8"
date:   2024-05-06
categories: 
---
Welcome back to my second post about [Hanukkah of Data](https://hanukkah.bluebird.sh/5784/). Please check out the first post for the background on this project and my approach to Nights 1-4. In this post, I'll go through Nights 5-8 and share my thoughts on the enitre puzzle. I'll continue to review and upgrade my original queries with the knowledge I gained.

### Night 5

Back to the Hanukkah grind! For this puzzle, I wrote out the prompt as: Person in Staten Island with old cats. From previous perusals of the `products` table, I knew that there were many cat food products where the description began with 'Senior cat'. I followed the same steps as Nights 2 and 4 to join my tables. Using `group by` was new to me and was helpful in counting the number of times each customer (phone number) purchased the specific products.

``` sql
select phone, citystatezip, count (phone)
from 
(select phone, customers.customerid, citystatezip, orderid 
from customers 
JOIN orders ON customers.customerid = orders.customerid)
as table1
JOIN

(select orders.orderid, customerid, desc
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku)
as table2

ON table1.orderid = table2.orderid

where citystatezip like 'Staten Island%' and desc like '%senior cat%'
group by phone;
```
This resulted in a table with 76 rows for each unique phone number and the number of orders for senior cat food. Like some of the previous nights, the query did not just return one phone number, but I quickly scrolled through the list and saw that most people had only purchased senior cat food once or twice. The outlier was someone who had 21 senior cat food purchases. I had found the person!

#### Upgrading my query
Surprise, surprise, I still hadn't figured out how to streamline my table joins. Here is the new and improved version. In this query, I combined the tables in 3 lines, then ordered by the count of phone numbers to quickly see the person who had bought senior food the most times.

```sql
select phone, citystatezip, count (phone)
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

where citystatezip like 'Staten Island%' and desc like '%senior cat%'
group by phone
order by count(phone);
```

### Night 6
This puzzle was quite difficult! I knew I had to compare three amounts- the amount the customer paid, Noah's sale price, and the wholesale cost of the items. The right person would be someone whose order cost was less than the other two numbers.

Here is what I landed on for my query. The first column was phone number, the second shows how much of a (percentage) discount the customer received. The third column was the total the customer paid, the fourth was the sum of each individual unit, and the final column was the sum of the items' wholesale cost.

```sql
select phone, ((1- (orders.total/sum(wholesale_cost)))*100) as column, orders.total, sum (unit_price), sum (wholesale_cost)
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

group by orders.orderid
having sum(wholesale_cost) > orders.total and sum(unit_price) > orders.total
order by column;
```
Finally an efficient join! I realized that two of my columns, `order total` and `sum(unit_price)` were the same amount. This makes sense, though I had thought `unit_price` was the sticker price that Noah's always sold the item at, instead of whatever discounted amount the customer paid. Still, my query worked and returned a list of people who got good deals at Noah's. There was one person on the list twice, both times with big discounts and that was the person I was looking for.

#### Upgrading my query
Now to review what I could have done differently. I rounded the percent discount to 2 decimal places and got rid of the unit price condition. Actually, when I ran this I got a way longer list of orders (still with the correct person with the highest discounts). I realized the error in the condition `sum(unit_price) > orders.total` from my original query. Since these numbers would almost always be the same (maybe slightly different with some rounding), comparing them filtered out a lot of purchases. I did get to the correct answer with my original query, but just by luck. Here is the optimized query:

```sql
select phone, round(((1- (orders.total/sum(wholesale_cost)))*100),2) as column, orders.total, sum(wholesale_cost), sum(unit_price)
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

group by orders.orderid
having sum(wholesale_cost) > orders.total
order by column;
```

### Night 7

I took a slightly inelegant approach to this puzzle. The challenge was to find the person who purchased the same item of a different color around the same time as the person from Night 6.

I started by poking around in the `products` table to see which items came in different colors:

```sql
select * from products where desc like '%blue%';
```
I tried a few different colors and noticed that all of the products with color options started with 'Noah's'. I then used the phone number from Night 6 to find out when the person bought these items:

```sql
select phone, name, desc, ordered
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

where phone like '585-838-9161' and desc like 'Noah''s%';
```
This resulted in 5 date options. I then wrote another query to find the people who bought a Noah's item on those days:

```sql
select phone, name, desc, ordered
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

where strftime ('%Y-%m-%d', ordered) in ('2018-12-31', '2020-02-01', '2020-06-28', '2021-10-07', '2022-04-23')
and desc like 'Noah''s%';
```
As you may have guessed, this query returned a lot of different purchases (105 in total). I got lucky in that I was scanning for 2 of the same item in a row and noticed the name of the Night 6 person quickly along with the person who bought the same item in a different color. If I hadn't seen the match immediately, I probably would have written 5 queries with the different `ordered` and `sku` values that I got from the previous query. Still, I found my Night 7 person!

### Night 8

The grand finale! This was a pretty easy puzzle. I had to find the person with a large number of Noah's collectible items. From the last puzzle, I knew the description I was searching for:

```sql
select phone, count(orders_items.orderid)
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN products ON orders_items.sku = products.sku
JOIN customers ON customers.customerid = orders.customerid

where desc like 'Noah''s%'
group by customers.customerid
order by count(orders_items.orderid);
```
There was one customer with a vastly greater number of purchases than everyone else in the list. That was the final person!

### Final Thoughts and Next Steps

After completing all 8 puzzles, I feel so much more confident in SQL. Once I figured out the efficient way to combine multiple tables, I could complete the puzzles a lot quicker and with better queries. Some of the puzzles were a little repetitive but I still feel like I got experience with many different SQL commands. I also feel more comfortable with how to research different actions in SQL since I was starting from a very minimal amount of knowledge.

Overall, I would highly recommend Hanukkah of Data! My boyfriend and I had a fun time solving the puzzles together and I learned a lot.