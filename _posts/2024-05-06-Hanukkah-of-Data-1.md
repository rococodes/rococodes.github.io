---
layout: post
title:  "Hanukkah of Data - Nights 1-4"
date:   2024-05-06
categories: 
---
Welcome to my first blog post. I am splitting my adventures with the [Hanukkah of Data game](https://hanukkah.bluebird.sh/5784/) into 2 posts. Read on for my approach to learning SQL through a game rather than a curriculum-based approach.

### Background - or - Why did I do it?

My main goal was learning SQL. I work with some data analysts on reports in my day job and wanted a better understanding of querying and pulling data. Since I have a BS in Physics, I’m familiar with some data concepts but I had never formally used SQL. My boyfriend and I were also doing long distance for a couple months and we wanted to have an activity to work on together while on videochat. He already knows SQL but wanted to brush up on his skills. Instead of doing step by step lessons, he suggested Hanukkah of Data to gamify the experience. Even though we didn't work on the game during Hanukkah, I actually liked having as much time as I needed to complete each puzzle and knowing that the game was complete. It is probably more fun to follow along with the game during Hanukkah, and I'm excited to do that this year.

### Getting started

I downloaded SQLite3 on my computer. I took a Python class in undergrad so I had worked a tiny bit in the terminal but that was several years ago at this point. I realized quickly that it was a bit tedious to type my SQL queries directly in the terminal so I opened up a Jupyter Notebook. This helped me keep track of notes and allowed me to change my queries more easily.

### Night 1

My first impressions were that I liked how the puzzle requires you to pull the data request from a little story/paragraph. It really helped me to rewrite the prompt into a short sentence, which I did for all of the nights. For this one I wrote: 'Phone number matches numberized name', where a numberized name takes a letter in the name and turns it into a digit (2 = ABC, 3 =  DEF, etc). 

When I first started this project, I thought I might use Python, which I actually did for the first puzzle. I had previously downloaded the data file from the Hanukkah of Data website, so I read the columns I needed- phone number and customer name into my Jupyter Notebook using a SQLite command.

``` python
con = sqlite3.connect("noahs.sqlite")
cur = con.cursor()
result=cur.execute("select name , phone from customers")
```

Then I created a list to store the (name, number) tuples. To make the process easier, at this point I made the names all lowercase with ‘.lower’. I also removed the ‘-’ from the phone number.

``` python
outcome=[]
for entry in result.fetchall():
    name=entry[0]
    number=entry[1]
    name=name.lower()
    number=number.replace("-","")
    outcome.append((name,number))
```

To check my work, I printed the first result from my list.

```
>>> ('jacqueline alvarez', '3153775031')
```

The next step was to check the condition from the puzzle: phone number = numberized name. Obviously it’s easier to go from name to number, since a letter can only have 1 corresponding digit but a digit could be 3 or 4 different letters. I wrote a function to do this, which was easier in Python than it would have been in SQL. It was pretty tedious since I had to be careful while writing 8 different if statements for each of the digits that correspond to letters. This function takes the name letter by letter and builds the numberized version.

```python
def numberize(name):
    numbername=''
    for letter in name:
        if letter in ['a','b','c']:
            numbername+='2'
        if letter in ['d','e','f']:
            numbername+='3'
        if letter in ['g','h','i']:
            numbername+='4'
        if letter in ['j','k','l']:
            numbername+='5'
        if letter in ['m','n','o']:
            numbername+='6'
        if letter in ['p','q','r','s']:
            numbername+='7'
        if letter in ['t','u','v']:
            numbername+='8'
        if letter in ['w','x','y','z']:
            numbername+='9'
    return numbername
```

Once I had my numberizing function, it was time to pass that over my (name, number) tuples and find a match 

```python
for entry in outcome:
    if (entry[1]) in numberize(entry[0]):
        print(numberize(entry[0]))
```

I ran this part and got a number!

```
>>>7268266362286
```

Uh oh, it’s longer than 10 digits (13, to be exact). Rereading the prompt, I realized it didn’t specify if the numberized name would be the full name, or just the first or last name. I decided to try the last 10 digits, assuming it would be the last name… nothing. I then tried again, reinserting the dashes from the original table. That worked and I officially passed Night 1!

Post script: In writing this recap, I also realized I could have just written a query in SQL to check the person’s name. I did that to verify that the name did appear in the customer table and that the customer’s first name was 3 letters.

``` sql
select name from customers where phone="826-636-2286"
```

I also could have printed entry[1], the phone number, instead of the numberized entry[0].
```python
for entry in outcome:
    if (entry[1]) in numberize(entry[0]):
        print(entry[1])
```

### Night 2

For the remaining nights, I switched to SQL. Just like Night 1, I pulled out the relevant information from the prompt: Find the person with initials JP who bought from the deli in 2017. For this puzzle, I knew I had to combine tables in SQL, which had not been necessary for the first night. This wasn’t too difficult of a join since both tables have customerid.

Here is what I wrote to join the customers and orders table:

``` sql
select phone
from customers 
JOIN orders ON customers.customerid = orders.customerid
where name like 'J% P%'
and strftime('%Y',shipped) like '2017';
```

When I ran this query I got 234 phone numbers! I reread the puzzle prompt and noticed another piece of information- the mysterious ‘JP’ bought coffee and bagels at Noah’s. I wrote another query to find the product information for said coffee and bagels.

```sql
select * from products where desc like "%coffee%";
select * from products where desc like "%bagel%";
```

There was only 1 row in the product table for coffee so I decided to use the sku for that item in my big query. I was also hesitant to use bagel since there could have been other products that were bagel sandwiches or otherwise labeled differently.

Now I had 3 tables I had to join- orders, customers, orders_items. My approach was to have 2 tables: the one I already pulled from my first query (joining date and name) and a second table with order ID and customer ID (where the order contained the coffee sku). I then joined those 2 tables together and pulled the phone number of the person with initials JP, who ordered in 2017, and bought coffee.

``` sql
select phone
from 
(select phone, customers.customerid, orderid 
from customers 
JOIN orders ON customers.customerid = orders.customerid
where name like 'J% P%'
and strftime('%Y',shipped) like '2017')
as table1

JOIN 
(select orderid, customerid from orders
where orderid in
(select cast (orderid as integer) from orders
INTERSECT
select orderid from orders_items where sku like 'DLI8820'))
as table2
ON table1.orderid = table2.orderid;
```

This query returned 1 phone number, which was the correct answer to Night 2. I’m writing this recap after finishing all 8 nights, and I’m cringing a little about how inefficient this query is. Still, this was the first time I had joined several tables so it was a good learning experience that I was able to improve upon as the puzzles continued. 

Now that I feel more comfortable with SQL, I would join orders to orders_items then customers to orders and then pull the phone number directly. This would be more efficient than creating 2 tables then combining them. Here’s how I would write it:

```sql
select phone
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN customers ON customers.customerid = orders.customerid

where name like 'J% P%' and strftime('%Y',shipped) like '2017' and sku like 'DLI8820';
```
This gives the same (correct) phone number and is a lot shorter, without needing intermediate tables and changing formats.


### Night 3

This puzzle required less SQL knowledge but more investigative skills, which I also really enjoyed. From the prompt, I noticed 3 pieces of information: the person is a Cancer (born between June 21 and July 22), born in the year of the Rabbit (1927, 1939, 1951, 1963, 1975, 1987, 1999, 2011, 2023), and lives in the same neighborhood as the person from the night before.

First step was to google the birth dates for Cancer and birth years for year of the Rabbit (filled in above). Then I wrote a quick SQL query to get the zip code of the person from Night 2.

```sql
select citystatezip from customers where phone like '332-274-4185';
```
Armed with my zip code, birthday date range, and birthday years, I wrote a query to the customers table.

```sql
select phone from customers 
where strftime ('%Y', birthdate) IN ('1927', '1939', '1951', '1963', '1975', '1987', '1999', '2011', '2023')
and
strftime ('%m-%d',birthdate) between '06-21' and '07-22'
and
citystatezip like '%11435%';
```
Even though I didn't need to do any joins for this puzzle, I was able to learn more about the strftime function and use outside research to inform my queries.


### Night 4

This puzzle had a fairly vague prompt, only specifying that the person bought pastries before 5am. I did some research in the product table and saw that I could use the sku BKY for bakery items.

```sql
select phone, ordered
from 
(select phone, customers.customerid, orderid 
from customers 
JOIN orders ON customers.customerid = orders.customerid)
as table1

JOIN

(select orders.orderid, customerid, ordered, sku
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid)
as table2

ON table1.orderid = table2.orderid
where strftime ('%H-%M', ordered) between '03:00' and '04:59'
and sku like 'BKY%';
```
Again, cringing at this long query to combine three tables that I could rewrite as such:

```sql
select phone, ordered
from orders
JOIN orders_items ON orders_items.orderid = orders.orderid
JOIN customers ON customers.customerid = orders.customerid
where strftime ('%H-%M', ordered) between '03:00' and '04:59'
and sku like 'BKY%';
```
Both of these queries returned the same list of 17 orders with phone numbers. I panicked for a second, since the previous puzzles had been straightforward with only one phone number. Going back to the original prompt, I saw that the woman we were looking for was often the first bakery customer, so I would expect to see her phone number multiple times. I took the number that appeared most frequently and that solved the puzzle!

### Wrapup

I hope this writeup is helpful or otherwise interesting. Rewriting my queries has allowed me to use my current level of knowledge to improve my initial attempts. It's a good reminder that there are many ways to approach a data/coding problem.

I would definitely recommend Hanukkah of Data for anyone looking to learn SQL. For brand new programmers, I think the nights are not in the best order for learning SQL. Puzzle difficulty can vary, rather than build from least to most complex. The game probably wasn't built for this exact purpose so I don't fault the creators at all. My other comment is that I would have liked the website to specify the format of phone numbers as 000-000-0000. I figured it out on the first night's puzzle, but it would have saved me a bit of time (and unnecessary debugging) had I known the formatting.

Be sure to read Part 2 of my Hanukkah of Data/SQL journey.
