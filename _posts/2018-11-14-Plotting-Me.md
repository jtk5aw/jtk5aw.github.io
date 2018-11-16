---
layout: post
title: Plotting Me
published: true
---

Here's my first post to this page! In it I'll talk about my first experience working with data in Python. The datasets that I dug into were based on my personal history using Goodreads.com, Instagram and Snapchat. Getting clean and detailed data about how I've used each of these was suprisingly easy. 

## Goodreads.com
The first dataset that I looked into was the one from Goodreads.com. It's content should come as no suprise. It was all about books. The data that was available was pretty minimal but that was mostly my fault since I hadn't been using the site for all that long. 

Of the data available to me I thought the two most intesting pieces were the number of pages in each book as well as the rating (out of 5) I'd given each book. Putting these together would allow me to visualize how much I've enjoyed reading. 

In order to make this visualization I had to find the sum of pages I'd read for each category. Thus I had to find a way to group the rating and number of pages for each book together. Once I did this, I just iterated through these pairs while keeping track of the pages read for each rating. The code I used to do all this is shown below.

```
ratingsAndPages = np.stack((myRatings, pages))
ratingPageSums = [0] * 5
for i in range(len(ratingsAndPages[0])):
  if ratingsAndPages[0][i] != 0:
    ratingPageSums[int(ratingsAndPages[0][i]) - 1] += ratingsAndPages[1][i] 
```

Now I had all the data organized and just had to plot it. The data lended itself to representation in bar and pie charts. Each of these are shown here and were created using matplotlib.

![Bar chart and pie chart of pages read by rating](/images/GoodreadsPlotsWithoutTitle.png)

Looking at the two charts showed promising signs! It seemed that I had been able to enjoy the majority of the time I spent reading. Well ... really it looked like I thought most of the books I'd read were average. So I guess the best way to say it is I *kinda* enjoyed the majority of the time I've spend reading.

## Instagram

I felt like I'd done the most interesting thing I could with the Goodreads.com data and decided to begin looking into the Instagram data I had. This was much more in depth and it seemed like I'd have a lot more options in deciding what I wanted to do with here. 

The first thing I decided to look into was my likes history since I'd made my instagram. This data was provided in a JSON file filled with every one of my past likes seperated into "media likes" which was just liked photos or videos and "comment likes" which was exactly what it sounds like. I had a lot more media likes so I decided to only consider those. I parsed the JSON file using the aptly named python package json and then put the media likes data into a pandas dataframe. 

Each like had a timestamp for when it occurred and what user's photo or video I had liked. I decided to focus only on the timestamp data and wanted to see how my likes were dispersed over my time on Instagram. I wasn't sure what method to use to most efectively show this. Since it was data that was changing over time my first thought was to just use a line plot. But this seemed too plain to me so I started looking at what other plots I might be able to use. I saw a really cool example of a heatmap and thought that would be a nice way to display this data. I could use one axis as the year and the other as the month; this would allow me to compare data in the same month from year to year as well as commpare months within the same year. Now I just had to start organizing the data. 

I saw that this was going to be trickier than the my reading plot for a couple of reasons. One, I couldn't use the timestamp data as it was and would to have to extract the month and year. Two, I would have to store the data based on two categories rather than just based on one in order for it to be plottable using the heatmap. To store the data I then had to create a 2d array with 12 rows (one for each month) and 6 columns (one for each year) that would track my likes for each month year combination. Once I had this in place, I then had to iterate through every row in my dataframe and take out the month and year from each timestamp. Then, by converting these extracted months and years into values between 0-11 and 0-5 respecitively I was able to find the cell in my 2D array that I would need to add a like to. All of this is in the code shown here 

```
yearAndMonthSums = np.zeros((12,6))
likedUsers = {}
for i in range(len(df2['Timestamp'])):
  # Putting Likes into 2D array with rows = Months and columns = years
  month = int(df2['Timestamp'][i][5:7])
  year = yearMap[df2['Timestamp'][i][0:4]]
  yearAndMonthSums[month - 1][year] += 1 
  # User counts for october
  if year == 4 and (month == 10 or month == 11):
    likedUser = df2['User'][i]
    if likedUser in likedUsers:
      likedUsers[likedUser] += 1
    else: 
      likedUsers[likedUser] = 1
sortedLikedUsers = sorted(likedUsers.items(), key=(lambda x: x[1]))
# Creates List of Colors to be used by October Likes Bar chart
zippedSortedLikedUsers = zip(*sortedLikedUsers)
colors = [assignColor(i[1]) for i in sortedLikedUsers]
```


## Snapchat
