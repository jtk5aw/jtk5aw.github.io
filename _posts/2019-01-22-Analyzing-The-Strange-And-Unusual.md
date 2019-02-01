---
layout: post
title: Analyzing the Strange and Unusual
published: true
---

---
With a little more experience writing these posts hopefully this is an improvement on the last. I'm going to talk about another data science project that I worked on; this time based on one of my favorite past times. I'll be analyzing the scripts for the podcast "The Bright Sessions". Definitely check it out if you haven't heard of it before! It just so happens that they post all of their scripts on their website so I thought I just had to use the data science skills I've been working on to dig a little deeper into what the show is about.

# Scraping

In order to get started with this project I first had to extract the script text from "The Bright Sessions" [website](http://www.thebrightsessions.com/). Looking at all of the episode scripts I saw that they all had a very similar URL structure. Each was simply www.thebrightsessions.com/episode-#. This would make it very easy to generate a URL for each individual script. With these URLs, it would then be possible to use python's requests library to extract the HTML that made up each scripts webpage. I did this using the following method that took in the current episode number as the string eps:

```python
def getRequestContent(eps):
    result = requests.get(URL+eps)
    while result.status_code != 200:
        time.sleep(10)
        result = requests.get(URL+eps)
    print("Sucessfully recieved " + eps)
    return result.content
```
**Note:** After a few requests there would be a non-successful status code returned. To fix this, I added the while loop that waits for 10 seconds and then tries again.

Now that I had the HTML of the whole webpage I had to parse out just the script text. Using some hackerman esque skills (read: inspect element) I was able to see that the HTML of the website was structured in a way that made this simple. Using BeautifulSoup, all that would be necessary was finding a block of HTML with the id "content" and class "main-content". Within this block was all of the script-text. Even more conveniently, each line of the script was within its own set of <p> tags (Well, kinda. More on that later). So in order to extract each line all that was necessary was to go through each individual set of <p> tags in this block. Then, each of these lines were appended to a list after being normalized (I'll be honest, not 100% sure what it does but it made analyzing the text later on possible). The method to do this is shown below:

```python
def getLines(c):
    soup = BeautifulSoup(c, features="lxml")

    content = soup.find("div", {"id": "content", "class": "main-content"})
    paragraphs = content.findAll("p")

    lines = []
    for line in paragraphs:
      lines.append(unicodedata.normalize("NFKD", line.get_text()))
    return lines
```

I then added the list that was returned by this method to a dictionary with the episode number as a key. Once all of the episodes were scraped, this dictionary was written to a JSON file so it could be easily accessed later on during the next part of the project. Organizing the data in this way also meant that the text didn't have to be re-scraped every time I wanted to work on the project and instead was only re-scraped when I wanted to update the script data.  

There was only one other hiccup in scraping the script text that I haven't mentioned yet; episode 17 was divided into two parts. This meant it was broken up into two separate URLs designated by 17a and 17b. This was a simple fix but gave me some trouble in the beginning when the requests would suddenly just stop working completely at 17. Both of these scripts were combined into one under the key 17.

# Cleaning

With all the data I needed now stored on my computer I could begin cleaning it up. My end goal was to have a dataframe with each spoken line as a single data point with the name of the speaker, the episode it was spoken in, its polarity and its subjectivity all associated with it.

Looking at the data it was possible to see that a majority of the lines were of the form "speaker" : "line spoken". So, I could go through line by line and split on the ":" to find the speaker and what they said. This seemed simple enough so naively I just went for it thinking it would work for the whole show. The code to do so is shown here:

```python
speaker_spoken = line.split(':')
speaker = speaker_spoken[0].strip()
spoken = ' '.join(speaker_spoken[1:])
```
 Does any body have a guess as to if that worked? Anyone? It didn't. Shocking I know, but even though a lot of the lines appeared to be structured that way there were also a lot that weren't. Go figure.

The first problem I noticed was the presence of effect lines. These lines were structured as follows [type of effect: the effect]. To find them I just had to find lines starting with "[type of effect". There were 9 different versions of "type of effect" so this meant searching through an array for every line to see if it started with any one of them. Code to do this is shown here:

```python
effect_list = ['[sfx', '[credits', '[tfx', '[passing',
              '[elevator', '[music', '[fade out', '(time', '[time']
for effect in effect_list:
          # If the line is just an effect we break
            if line.startswith(effect):
                is_effect = True
                to_be_df.append(('effect', episode, line))
                break
```

Next on the list of issues, I saw that there were actually certain spoken lines that were divided on the webpage between multiple sets of <p> tags. This meant that when I would attempt to associate this line with a speaker, there would be none as there would be no ":" to split on. To fix this, any line that was over a certain length (the length of the longest [sfx] line) and didn't have a ":" was just appended to the string in the previous index of the array. There were some other special cases I had to consider when doing this as well but in the end all of the text was attributed to the proper speaker.

Well, almost the proper speaker. There were a few cases of slight differences in how the names were displayed on the webpage (e.g an extra space, Dr instead of Dr. etc) that led to them being recognized as different characters. This was an easy fix as there were so few of them that I could just manually change what I needed to.

Lastly, there was one very specific fix I had to make in the first episode. An effect was embedded within spoken text rather than being its own separate line. Since this did only occur once the effect was pulled out of the text on the specific line and made it's own data point.

With all of that handled, I now had all the text attributed to a speaker as well as knew what episode it was from. The only two things left that I was hoping to add to the data frame were the polarity and subjectivity values of the text. I decided to do this using TextBlob. Before I passed the text through TextBlob I first removed all punctuation and made all letters lowercase. This would make it easier for TextBlob to recognize certain words and process them correctly.

It's worth noting that TextBlob doesn't provide the most sophisticated method of calculating polarity and subjectivity but it would work for my purposes. At the end of this post I'll talk about some of the downfalls of using it.

# Graphing

Now that I had all the data I wanted together in one data frame I could start working to visualize it.

First, I wanted to look at the overall polarity of the show by comparing the number of lines with positive and negative polarities. A pie chart would have just been too boring so I went with something new, a Waffle Plot. It was something I'd seen used in some other projects and had caught my eye (also has a pretty cool name). Now was as good a time as any to try it out.

![Waffle Chart of Positive and Negative Polarity Lines Spoken](/images/PositiveVsNegativeWaffleChart.PNG)

What the I saw wasn't what I expected; I thought there would be an almost even number of positive and negative lines. This piqued my interest so I thought next I could make a similar comparison but this time by character. This time, the best plot I could think of was a Box and Whisker plot as it could show the spreads of each character side by side.

![Box and Whisker Plot of Polarity By Character](/images/PolarityByCharacter.PNG)

All the characters had almost identical spreads of polarity. None of them were significantly different than any other and all of them leaned towards being more positive than negative. This was again a surprise as I thought at least one character would be significantly different from the rest.

From here I wanted to move on from polarity and decided to look into subjectivity. Since subjectivity is very similar to polarity I thought another Box and Whisker plot would be my best bet again.

![Box and Whisker Plot of Subjectivity By Character](/images/SubjectivityByCharacter.PNG)

Again no character showed any significant difference from the rest. This wasn't as much of a surprise to me as subjectivity didn't seem like it would vary as much based on the content of the show. Since it was a fictional show having each character lean towards expressing opinions rather than facts would make sense.

Before wrapping up this project there was one more plot I wanted to experiment with. Recently I'd seen a lot of plots consisting of one square with smaller squares within it whos area represented the percentage some characteristic made up of the whole. I don't really know what these plots are actually called but I really liked the look of them. To incorporate one into this project I found how many lines were spoken by each character and plotted that in this "Square" plot (for lack of a better name).

![Square Plot of Lines Spoken By Character](/images/SpokenLinesByCharacter.PNG)

# Conclusions

When I was looking for a library that would allow me to perform Sentiment Analysis TextBlob was by far the easiest to use. However, it was also the least sophisticated and because of that probably the least accurate. All TextBlob does is query a stored database of words for every word it comes across to see if it has some polarity/subjectivity values or multipliers associated with it. Because of this word by word process of measuring the text, TextBlob has difficulty understanding context. The multipliers work to combat this since they are tied to words like "not" or "very" and change the polarity/subjectivity values of whatever words follow. However, it still leaves a lot of room for improvement. In the future, I could return to this project and try using a more sophisticated algorithm for Sentiment Analysis in order to see if it would lead to clearer or different results. 
