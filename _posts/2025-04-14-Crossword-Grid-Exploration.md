---
layout: post
title:  "Crossword Grid Exploration"
date:   2025-03-04
categories: 
---

It's been a while, loyal readers. I've actually been working on several posts in my NYT crossword series so hopefully those are now available for you to read through. Unlike the other posts in the series, this project is not related to my own crossword solving results. Instead, I am looking at grid designs (where the constructor chooses to place black squares in a puzzle).

### It all starts with Reddit

I'm not a big social media person, but the inspiration for this project came from [a post I saw on the crossword Reddit page](https://www.reddit.com/r/dataisbeautiful/comments/7d7u2v/heatmap_of_nytimes_crossword_grids_by_day_of_week/?sort=new#lightbox). Intrigued, I looked in the comments for a link to the creator's GitHub project so I could get some inspiration from his approach. A kind Redditor had already asked to see the code and received a response to the effect of 'this took 30 minutes and was so trivial even a child could figure it out'. Alright then, challenge accepted! 

### The importance of crossword grids

Before I get into any programming, I have to give some background on why this effort is so interesting to me. There are a few golden rules for the NYT crossword (which are, of course, sometimes broken). 
1. Grids from Monday - Saturday are 15x15, while Sunday is 21x21
2. Puzzles must follow rotational symmetry (ex. a black square in the top left quadrant will correspond to a black square in the bottom right quadrant)

This knowledge, combined with the fact that puzzle difficulty increases throughout the week, leads to the patterns you can see in the Reddit post. The early days of the week follow a more standard structure, reflected in the dark squares around the edges of the puzzle.

I sometimes construct my own crosswords, so this is also helpful to really understand how constructors choose their grid designs.

### Using the New York Times API

For a more in-depth look on how to use your NYT subscriber token to make API calls, see my [earlier post](https://rococodes.github.io/2024/05/13/NYT-Crossword1.html).

As detailed in the other post, there are 2 API calls I need to make. The first one takes an input of a date range (up to 100 days) and provides the following keys:
```
'author', 'editor', 'format_type', 'print_date', 'publish_type', 'puzzle_id', 'title', 'version','percent_filled', 'solved', 'star'
```

The second API call uses ```puzzle_id``` as input and gives these response keys for a particular puzzle:
```
'board', 'calcs', 'firsts', 'lastCommitID', 'puzzleID', 'timestamp', 'userID', 'minGuessTime', 'lastSolve'
```

For this project, the only key I care about is ```board```. Here is a sample of the first 15 elements (first row of a crossword):
```
    "board": {
        "cells": [
            {"guess": "T",
                "timestamp": 1709214417},
            {"guess": "I",
                "timestamp": 1709213663},
            {"guess": "P",
                "timestamp": 1709213668},

            {"blank": true},

            {"guess": "I",
                "timestamp": 1709214657},
            {"guess": "N",
                "timestamp": 1709214425},
            {"guess": "A",
                "timestamp": 1709214728},
            {"guess": "T",
                "timestamp": 1709214738},
            {"guess": "U",
                "timestamp": 1709213682},
            {"guess": "B",
                "timestamp": 1709214884},

            {"blank": true},

            { "guess": "G",
                "timestamp": 1709213581},
            {"guess": "A",
                "timestamp": 1709213582},
            {"guess": "Z",
                "timestamp": 1709213582},
            {"guess": "A",
                "timestamp": 1709213582},
```

As you can see, the first row of this grid goes- 3, **1**, 6, **1**, 4. Since this a puzzle I have solved, my guesses will appear in the results, with a blank corresponding to a black square.

### My programming approach

For the final heatmap graphic, I don't care about the actual letter that goes in each white square. That means that I first have to get all of the grid values into a list, then 'clean' that list so it only has 2 options- white or black square. Let's do that first:

```python
index = 0
grid_diagram = []

def get_board_squares(board):
    for index in range(len(board)):
        if 'guess' in board[index]:
            grid_diagram.append(board[index]['guess'])
        else:  
            grid_diagram.append(' ') 
```
This produces a solved grid diagram. At this point, I could also write these solved grid diagrams to a csv file as well. 

```python
def write_to_csv(grid_squares, print_date, dotw):
    filename = f"[yourfilepath]/{dotw}/{print_date}.csv"
    with open(filename, 'w') as csvfile:
        csvwriter = csv.writer(csvfile)
        csvwriter.writerows(group_rows(grid_squares))
```
The file path creates separate folders for each day of the week and titles the csv with the puzzle print date. The ```group_rows``` function called in the last row is:

```python
def group_rows(list):
    return[list[i:i + 15] for i in range(0, len(list), 15)]
```

This function actually isn't that good, can you see why? To have the csv match the original grid format from the list that we get from the ```get_board_squares``` function, the puzzle has to have a dimension of 15x15. This was fine for most Monday-Saturday puzzles, but the files would be off for all Sunday puzzles and the rare nonstandard puzzles earlier in the week. I mainly chose to ignore puzzles outside of the standard dimension, and you can see how in my final code later in this post.

Ok, now we have to clean the grid to just white and black squares:

```python
cleaned_grid = []
def clean_grid(grid_squares):
    for i in grid_squares:
        if i == ' ':
            cleaned_grid.append(1)
        else:
            cleaned_grid.append(0)
```
This one is pretty self explanatory. The function creates a new list (which will be 225 elements long, from a 15x15 grid) and appends it based on the input so that a black square is a 1 and a white square is a 0. This sets up well for heatmap math later on, since squares with a higher likelihood of having a black square will have a higher numerical value.

```python
crossword_matrix = []
def group_grids(list):
    arr = np.array(list)
    cleaned_grid_list.append(arr)
    
    list.clear()
    grid_diagram.clear()  
    cleaned_grid.clear()
```
Before running the above function, I have a long list with 1s and 0s. The first two lines turn that list into an array, then add it to a list of arrays. Before this point, I am working with just the one puzzle/grid information from my for loop, but I do need to combine them all as elements to create the heat map. Eventually I will need one single 'matrix' that is 15x15 (just like how a standard grid is) to feed into the heat map, but that step comes later. For now, I'm ending up with a list of arrays.

I also have to clear out some of the other lists I'm using so that they don't get appended with each new puzzle. I honestly don't know if this is even the best way to do it, but it works to 'reset' those lists. This took a lot of effort to figure out, and I still don't completely understand how I'm creating global lists but using them inside functions and then clearing them in other functions. Oh well.

Anyway, now I'm ready to put these little functions into my big function that performs the API call. For more information, see my first post in the crossword series. I then nest in another for loop, where I exclude all of the irregularly-sized puzzles. For Sundays, I would replace 225 with 21x21=441.

``` python
 if len(board) == 225:
                get_board_squares(board)
                clean_grid(grid_diagram)
                group_grids(cleaned_grid) 
            else:
                continue
```
So what do I end up with? A list called ```cleaned_grid_list``` populated with a 225 element list from each puzzle where black squares are 1s and white/blank squares are 0s.

Getting this data at scale is very tedious since the API will only let you request information on 100 puzzles at a time. You can read the function in more detail in my other post, but I also specify the day of the week as an input for the final function to only get data from puzzles published on one day of the week at a time.

The final part of the code is below. As you'll see below, I add all of the puzzle-lists with the ```matrix_math``` function

```python
def matrix_math(list):
    matrix_sum = [0]*225
    for i in list:
        matrix_sum = np.add(i, matrix_sum)
    
    print(matrix_sum)

    pandas_and_graph(matrix_sum)
```

To do the matrix addition (adding each element to the element in the corresponding spot), I create an empty list that is 225 elements long. I iterate over ```cleaned_grid_list``` to add the elements of each additional puzzle to that new list. This also prints out the sum, which appears like this, still not the right 15x15 dimensional form:

```
[  4   1   1  43 144 107  19   4  19 104 146  53   0   0   2   1   0   0
  32 135  95  11   1  15  88 134  39   0   0   1   1   0   0  12  58  33
   3   0  13  59  92  28   0   0   1  49  31  18  49  47  53  65  57  64
  65  39  41  27  53  75 132 120  86  59  23  32  68 109  70  25  21  41
  55  87 105  97  76  63  47  30  29  60  95  54  28  21  42  49  77  94
  35  29  27  62  75  95  69   3  70  72  78  68  27  33  41  52  30  17
  54  80  56   6  37   6  56  80  54  17  30  52  41  33  27  66  77  73
  70   4  69  96  74  60  27  29  35  95  77  49  43  21  29  53  94  59
  30  30  48  63  76  98 103  86  53  41  22  26  71 109  69  33  24  59
  84 119 130  74  53  28  42  40  64  62  56  63  52  48  50  19  31  48
   1   0   0  26  94  59  13   1   3  33  60  10   0   0   1   1   0   0
  37 135  88  15   2  11  95 136  30   0   0   1   2   0   0  51 146 105
  19   5  19 108 144  41   1   1   4]
  ```

To do that, we'll look back at ```matrix_math```, where I call a function called ```pandas_and_graph```. Here's that function:

```python
def pandas_and_graph(list):
    arr = np.array(list)
    matrix = arr.reshape(15,15)
    crossword_matrix = pd.DataFrame(matrix)
    sns.heatmap(crossword_matrix, linewidth=.5, cmap = "hot_r")
```
I'm not really sure why I have to put the matrix sum into an array again, but the whole program would get messed up if I removed that np.array line that's the same in the ```group_grids``` function. It also took some trial and error to figure out that the heat map matrix needed to be in a pandas dataframe to feed into the heatmap module. I used the Seaborn module, and you can find [the documentation here](https://seaborn.pydata.org/generated/seaborn.heatmap.html). There are lots of parameters you can customize. I chose to add in borders around each square to mimic a crossword grid. I also used the same color map that the original Reddit poster used, but there are a variety to choose from.

### Final heatmaps for each day of the week
You might think it would be simple to actually run this program on the grids once I completed the code for receiving and analyzing the data. Interestingly enough, working with the New York Times API wasn't as straightforward as I had hoped. In my past crossword blogs, I was working with my own data to understand my progress in solve times. That is, I was limited to analyzing puzzles I had completed as part of my crossword streak. However, the crosswords online start from November 1993. That gave me high hopes of getting grid data for over 30 years of puzzles. I even wrote some functions to automatically feed my program the begin and end dates for requests (since the API is limited to 100 puzzles per call). When I ran that, I found out that if I haven't started a puzzle, I am not able to pull the grid structure through the API. As of today, I've solved 1868 puzzles so I'm satisfied that I have a decent sample size for the purpose of this investigation. 

I'll need to add an additional condition in my code to skip any puzzles that I haven't solved but that is pretty easy.

This blog post is pretty long, so I think I'm going to end here, with a bit of a teaser/cliffhanger. If you want to see all of heat map results and my lessons learned, come back for my next post.

![monday grid](/assets/images/mondays.png)