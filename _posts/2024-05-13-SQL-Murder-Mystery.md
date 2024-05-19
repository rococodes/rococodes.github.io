---
layout: post
title:  "SQL Murder Mystery"
date:   2024-05-13
categories: 
---
As a palate cleanser to my Python activities, I want to continue exploring the world of SQL games. A classic in the genre is [the SQL Murder Mystery](https://mystery.knightlab.com/). I have enjoyed using SQL games to improve my skills and recommend them to anyone learning the language. Playing a game really helps me get a feel for working with different data structures and requests. I also like how it requires some creative thought to interpret clues outside of simply writing queries.

If you don't want spoilers, I'll give my thoughts on the puzzle before I walk through my queries and the results.


### Intro and Thoughts

I think this would be a great initial introduction to SQL. The SQL interface works intuitively on the website, so you don't have to download a file and run queries in the terminal. The drawback of that is that the interface doesn't save past queries after you reset. I like to keep a separate note of what I've already queried in case I need to rewrite an input. It was helpful for the game to provide the schema diagram so that you can visualize how all of the different tables work together. I also liked how the website linked a bunch of different resources, including a [SQL walkthrough](https://mystery.knightlab.com/walkthrough.html) that is pretty thorough.

I'll get into this more below, but if you're already comfortable with SQL, this might not be the game for you. The joins and queries are fairly straightforward, and while the story was fun, I was left a bit underwhelmed by the result. That is, I was expecting a couple more twists and turns for a mystery investigation. Still, I thought this was a fun way to spend 30-45 minutes and I would definitely recommend the SQL Murder Mystery for someone learning SQL.

### My Solution

As a reminder, there are many ways to get to an answer when querying databases. This is just my thought process and you could split the queries into multiple steps and still get the same answer. In the same vein, I'm sure there are SQL experts reading this and thinking of ways to make my queries more efficient. If you're brand new to SQL, I might even suggest you write more queries (that might be less streamlined) to practice the notation. Now let's get investigating!

Our mission is to find who committed the murder. We start with just a small piece of information- the murder took place on 1/15/2018 in SQL City.

To see the format of data in the table, you can fire off a quick query like:
```sql
select * from crime_scene_report limit 2;
```
In this case, the query was helpful since dates can be stored in different ways and I wouldn't have necessarily phrased my query as ```YYYYMMDD``` at first. With that knowledge, here is the specific query to find the report of our murder:

```sql
select * from crime_scene_report 
where date = '20180115' 
and city = 'SQL City' 
and type = 'murder';
```
```
Security footage shows that there were 2 witnesses. 
The first witness lives at the last house on "Northwestern Dr". 
The second witness, named Annabel, lives somewhere on "Franklin Ave".
```

Let's start on the first witness. To be more direct, I'm going to join the interview table to the person table on person ID to get the interview text in one query. If I didn't want to join, I could query the perosn table first to get ID, then query the interview table.

 I'll add a condition for the street name and order by house number since we know that will be the highest in the system.

```sql
select name, address_number, transcript from person 
join interview on person.id = interview.person_id
where address_street_name like "Northwestern Dr%"
order by address_number desc;
```

| name      | address_number  | transcript |
| ----------- | ----------- | ---|
| Morty Shapiro      | 4919       | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |

Everything looks to be in order. Now I'll move to the second witness:

``` sql
select name, address_strees_name, transcript from person 
join interview on person.id = interview.person_id
where address_street_name like "Franklin Ave" and name like "Annabel%";
```
| name      | address_street_name  | transcript |
| ----------- | ----------- | ---|
| Annabel Miller      | Franklin Ave       | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |


```sql
select person.name, transcript from person
join drivers_license on drivers_license.id = person.license_id
join get_fit_now_member on get_fit_now_member.person_id = person.id
join interview on interview.person_id = person.id
where get_fit_now_member.id like '48Z%' 
and drivers_license.plate_number like '%H42W%';
```
Looking back over this, I'm realizing I only used the information from the first interview! Only one person came up in the query so I'm assuming that's the right individual but let me just cross check really quickly: 

```sql
select * from get_fit_now_member 
join get_fit_now_check_in on get_fit_now_check_in.membership_id = get_fit_now_member.id
where check_in_date like '20180109'
and person_id like '67318';
```
I got the same person so that's great. Here's their interview:

| name       | transcript |
| ----------- | ----------- |
|  Jeremy Bowers | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

At this point, I actually made a mistake! Since I was following the trail of clues set up in the interview, I wrote another query to find the woman mentioned. However, you were actually supposed to check the solution at this point. If I had done that, I would have seen this message:

 solution       | 
| ----------- | 
| Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer. |

But I just went right ahead to find the criminal mastermind. I guess that's just my inquisitive spirit but maybe this is something the puzzle could have specified. From a legal standpoint, the hirer is definitely also responsible. But that's not so relevant to the SQL side of things so back to the query: 

```sql
select * from person
join drivers_license on drivers_license.id = person.license_id
join facebook_event_checkin on facebook_event_checkin.person_id = person.id
where car_make = 'Tesla'
and hair_color = 'red'
and height between 65 and 67
and event_name like '%symphony%';
```
This one actually gave me a bit of trouble because I was trying to outsmart the puzzle. I thought I would have to find an interview and untangle a big conspiracy, however when I joined the interview table I wasn't getting any results. That was because this person wasn't interviewed since the game ended there. I also found that I didn't need all of the information from the previous interview. This could have been intentional or could have just been a result of limited data in the database.

I put the one result into the solutions checker and that was that!

 solution       | 
| ----------- | 
| Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!|

### Final Thoughts

If you found this post while you're looking for help with the SQL Murder Mystery solution, I hope it was helpful. I am planning to continue my reviews of different free SQL games and put together a roadmap for anyone who wants to learn SQL through games. This game would be a great first step on a SQL learning journey because of the relatively simple queries and SQL intro on the website. Enjoy and let me know what you think of the game!