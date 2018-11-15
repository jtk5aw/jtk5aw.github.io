---
layout: post
title: Plotting Me
published: true
---

Here's my first post to this page! In it I'll talk about my first experience working with data in Python. I worked with data about my usage of the social medias Goodreads, Instagram and Snapchat. Getting clean and detailed data about my personal history on each of these was suprisingly easy. 

## Goodreads.com
The first dataset that I looked into was the one from Goodreads.com. The content of this data should come as no suprise. It was all about the books. The data that was available was pretty minimal but that was mostly my fault since I hadn't been using the site for all that long. 

Of the data available to me I thought the two of the most intesting pieces were the number of pages in each book as well as the rating (out of 5) I'd given each book. Putting these together I thought it would be possible to see how much of my reading I've enjoyed.

To visualize this I found the sum of pages I'd read for each category. This wasn't too difficult as all I needed to do was group the rating and number of pages for each book together and then keep track of the pages read for each rating. The code to do this is shown below
~~~~~
ratingsAndPages = np.stack((myRatings, pages))
ratingPageSums = [0] * 5
for i in range(len(ratingsAndPages[0])):
  if ratingsAndPages[0][i] != 0:
    ratingPageSums[int(ratingsAndPages[0][i]) - 1] += ratingsAndPages[1][i] 
~~~~

Now I had all the data I needed and just had to plot it. The data lended itself to representation in bar and pie charts. Each of these are shown here and were created using matplotlib.

![Bar chart and pie chart of pages read by rating](/images/GoodreadsPlotsWithoutTitle.png)

Looking at the two charts showed promising signs! It seemed that I had been able to enjoy the majority of the time I spent reading. Well ... really it looked like I thought most of the books I'd read were average. So I guess the best way to say it is I *kinda* enjoyed the majority of the time I've spend reading.

## Instagram

## Snapchat