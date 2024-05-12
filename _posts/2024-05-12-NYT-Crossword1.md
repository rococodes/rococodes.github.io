---
layout: post
title:  "Visualizing Crossword Data"
date:   2024-05-12
categories: 
---

Welcome to my series on using the New York Times Crossword API with Python to get my past crossword data.

### What is the NYT Crossword?

Before I get into the code, I'll explain some background on crossword puzzles. Lots of outlets publish crosswords, but the New York Times crossword is probably the most iconic of the daily crosswords. Monday-Saturday crosswords are on a 15x15 grid and get progressively harder over the course of the week. The Sunday crossword is on a 21x21 grid and is said to be on the same difficulty level as a Wednesday/Thursday. Monday-Thursday and Sunday crosswords have themes, where several long answers go together and often have a punny connection. If you're nerdy enough to read this post (complimentary), you might already be a seasoned solver, but if not, check out [this blog post](https://www.nytimes.com/article/how-to-solve-a-crossword-puzzle.html) on getting started with solving.

### Using the NYT API

Since this was my first time working with an API I wanted to devote a blog post just to that topic. In future posts, I'll focus on working with the data I pulled.

This project was an interesting introduction because I had to make two different API calls, one for public data and one for data that required an authentication token. Here I will shout out [this GitHub project](https://github.com/kesyog/crossword/tree/main) from kesyog which helped me a lot. Their program does not use Python for the API call so I was partially on my own. I decided to work on this project in a Jupyter Notebook since I like the format for viewing data and I'll want to plot my results at some point.

#### API Call 1 - public crossword data

The first API call gets information on each daily crossword. This is a pretty straightforward section of code that you can see below. I'm using the Python `requests` module and the URL from kesyog's code. The one wrinkle to keep in mind is that the NYT site will only let you pull 100 days of crosswords with each call. I'm going to figure out how to deal with that later since I'm eventually going to want several years of data. For now, I just want to make sure that I can get the data I need.

```python
url = "https://www.nytimes.com/svc/crosswords/v3/36569100/puzzles.json?publish_type=daily&date_start=YYYY-MM-DD&date_end=YYYY-MM-DD"
headers = {"accept": "application/json"}

r=requests.get(url, headers=headers)
print(f"Status code: {r.status_code}")
response_dict = r.json()
print(response_dict.keys())
```
Here I am calling the information for a date range and putting it into a Python dictionary, keeping the same keys as the original request. I don't know what those keys are, so I want to print those, as well as the status of the request. The output is below:

``` python
Status code: 200
dict_keys(['status', 'results'])
```

A status code of 200 means that the request was successful. The keys are ```status``` and ```results```. Status was just ```OK``` and the real information was in ```results```. Knowing that all of the information I need is in that dictionary, I wrote a query to put the items from the ```results``` dictionary into a new dictionary. First the program will return the number of items in the dictionary (the number of crosswords requested, limited at 100 from my original call). Then it will print the keys in the results dictionary by looking at the first entry in the dictionary:

```python
repo_dicts = response_dict['results']
print(f"Repositories returned: {len(repo_dicts)}")

repo_dict = repo_dicts[0]
print(f"\nKeys: {len(repo_dict)}")
for key in sorted(repo_dict.keys()):
    print(key)

print(repo_dicts[0])
```
This is the outcome. I also printed an example so we can see how the data is filled out for one date.

```
Repositories returned: 100

Keys: 11
author
editor
format_type
percent_filled
print_date
publish_type
puzzle_id
solved
star
title
version

 {
    "author": "Harry Zheng",
    "editor": "Will Shortz",
    "format_type": "Normal",
    "print_date": "2024-01-01",
    "publish_type": "Daily",
    "puzzle_id": 21588,
    "title": "",
    "version": 0,
    "percent_filled": 0,
    "solved": false,
    "star": null
        }
```
This all looks pretty good. ```puzzle_id``` and ```print_date``` will definitely come in handy later. Now time to get my crossword data.


#### API Call 2- My results

For this part, you'll need to have an active NYT crossword/games subscription and your subscription token. I followed the instructions on [the GitHub project I mentioned above](https://github.com/kesyog/crossword/tree/main) to get mine. 

```python
url = "https://www.nytimes.com/svc/crosswords/v6/game/{puzzle_id}.json"
cookies = {'NYT-S': 'yourtokengoeshere'}
headers = {"accept": "application/json"}
c=requests.get(url, headers=headers, cookies=cookies)
crossresponse_dict = c.json()
print(crossresponse_dict.keys())
print(crossresponse_dict)

dict_keys(['board', 'calcs', 'firsts', 'lastCommitID', 'puzzleID', 'timestamp', 'userID', 'minGuessTime', 'lastSolve'])
```

The only new item in this call is the ```cookies``` line, which is how you access your personal crossword data. Also note that you have to enter the puzzle ID in order to get the solve information. I printed the entire result since I didn't know the format of the data beyond the keys. ```board``` is a record of every single letter guess entered into the grid so I'll skip printing that. The other results are below:

```
'calcs': {
    'percentFilled': 100, 
    'secondsSpentSolving': 151, 
    'solved': True}, 
'firsts': {
    'opened': 1704087857, 
    'solved': 1704088008}, 
'lastCommitID': '6A8DFF6', 
'puzzleID': 21588, 
'timestamp': 1704088009, 
'userID': [removed], 
'minGuessTime': 1704087858, 
'lastSolve': 1704088009
```
There is a lot of information, but for my purposes I'll only need to use ```secondsSpentSolving``` from this data.


### Using the data

I hope this post was helpful to understand how the New York Times crossword API works. As a next step, I'll need to update some of my code so I can easily pull useful data for my plots. Still, I wanted to have this post introducing working with the API since it's new to me.

