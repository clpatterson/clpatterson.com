---
title: "Using Regex for Messy Webscraping"
date: 2018-07-12T15:36:17-04:00
draft: false
---


People have strong feelings about using regex for parsing html. I understand. When things are as they should be, scraping html using xpaths or css selectors is way better. Python packages like lxml and beautifulsoup do a great job. But things aren’t always as they should be…

![](img/titles_with_divs.jpg)

or

![](img/titles_with_ptags.jpg)

Div and p tags between h2 tags is not cool. But what’s less cool is horribly inconsistent html across the posts I want to scrape. Xpaths and css selectors just won’t be able to get the job done. Enter regex.

With regex I can latch onto what structure exists higher in the DOM tree and flexibly grab content that lives below, in the muck. The key to doing this well is not letting your regex get too complicated.

```python
re.findall(re'>.*?(([0-9]{4}\.[0-9]{2}\-\-[0-9]{4}\.[0-9]{2}\s.*?|[0-9]{4}\.[0-9]{2}\—[0-9]{4}\.[0-9]{2}\s.*?|[0-9]{4}\.[0-9]{2}\-\-\s.*?|[0-9]{4}年.*?))<', higher_level_element)
```

I started crafting this expression to account for date variations at the beginning of strings I wanted to scrape. Then I stopped myself. If you write something like this, chances are you will have no clue what it does in a few months time. It’s always easier to grab more than you need then filter down to your desired text. I start by grabbing everything between two elements I know won’t change.

```python
title_tag = re.search(r'<h2 class="tit">(.*?)</h2>', body, re.S)
title_tag = title_tag.group()
title = re.findall(r'>([\u4e00-\u9fff].*?)<',title_tag)
```

Here `body` is the whole http response and `re.s` is an abbreviated version of the DOTALL flag. DOTALL allows the '.' wildcard to pickup newlines as well. `[\u4e00-\u9fff]` is the CJK unicode range for Chinese characters. This pattern does a great job for post titles I'm scraping. But for the body of posts where text can start with dates, this doesn't do the job. There I use a pattern that will give me everything between ‘>’ and ‘<’.

```python
tag_values = re.findall(r'>[^>]*<',text_tag)

# Find Chinese text and clean.
text = []
for item in tag_values:
    han_check = re.search(r'[\u4e00-\u9fff]', item)
    if han_check != None:
	    clean_string = re.sub(r'[\u3000]?[\r]?[\n]?[\t]?[>]?[<]?', '', item)
	    clean_string = clean_string.strip()
	    text.append(clean_string)

```

All element values and spaces between elements are output by this pattern. In this case I’m looking for Chinese text, so I build out a few lines to test output matches for Chinese unicode. As you can see I also filter out what I don’t want with re.sub and clear out extra white space.

That’s it! Things can get more complicated than this, but strive I to avoid complexity in my regex. And be careful about using greedy expressions as a cure all. Regex ain’t the first choice for parsing html, but when things get salty it gets the job done.