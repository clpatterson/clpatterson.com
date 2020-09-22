---
title: "Deduplication with Fuzzywuzzy and Stanford CoreNLP"
date: 2018-03-21T15:55:54-04:00
draft: false
description: "Using open source tools to clean scraped data and eliminate duplicate Chinese Corruption reports."
---

Find and eliminating duplicates is an important part of data collection. For a few weeks now I’ve been helping an organization in New York collect official reports on corruption in China. Solving deduplication has been a key component of that effort. I’ve written this post to better understanding my own solution, but also in the name of sharing. I hope it’s helpful to others on their own self-taught journey.

### Background
Since Xi Jinping’s ascension to the Chinese presidency in 2012, he has waged a fierce anti-corruption campaign, removing corrupt officials and political foes alike. Apart from being a useful tool for consolidating his power, the campaign is also a source of political legitimacy for the Chinese Communist Party, which in recent years has been seen as increasingly corrupt by the Chinese people.

In order to show the people progress is being made in the anti-corruption effort, the Central Commission for Discipline Inspection (CCDI)--the central government office responsible for carrying out the anti-corruption campaign--and the provincial level Commissions for Discipline Inspection (CDI) publish details relating to individual cases of corruption on their respective websites. By scraping and aggregating these reports, scholars and journalists can more easily track and find patterns in the corruption campaign as it develops. So this New York organization took up the cause.

The case reports share a formal language and structure across all sites. Each post has a title, date, source, and body. The similarity makes for a great dataset, but frequent cross-posting between the CCDI and provincial CDI sites can create duplication.

To create a higher quality dataset, I needed to eliminate duplication.

### Deduplication Via Title Comparison
At first glance re-posted articles seemed to be identical across sites, so I tried a simple comparison of titles using sets.
```python
sichuan_vs_ccdi_title_matches = set(c_title) &amp; set(s_title)
print('matches: ' + str(len(sichuan_vs_ccdi_title_matches)))

>>> matches: 19
```
Cursory exploration of both ccdi and sichuan cdi data suggested many more matches. It was never going to be as easy as set comparison. Re-posted reports are almost entirely faithful to the original, but not quite. Slight variations make set comparison impossible (Chinese and English for your enjoyment):

* 四川省地质矿产勘查开发局党委常委祝世强接受审查
* 四川省地质矿产勘查开发局党委常委、副局长祝世强 接受组织审查
* Sichuan Provincial Bureau of Geology and Mineral Resources Party Standing Committee Member Shiqiang Zhu is under examination.
* Sichuan Provincial Bureau of Geology and Mineral Resources Party Standing Committee Member and Deputy Directory, Shiqiang Zhu is under CDI examination.

As you can see, the addition of a professional title or organization changes string length but not meaning. I needed something that would output a percent difference between strings. My roommate suggested SeatGeek’s open source Fuzzywuzzy library for string matching. And I was off!

### Fuzzywuzzy

Fuzzywuzzy is a simple api with 4 routines for fuzzy string matching. It’s routines match simple strings as well as out of order strings or strings with the same meaning, but different length, etc. Luckily my titles are highly formalized bureaucratic lingo, so word order or drastic differences in string lengths is not an issue. The simple string similarity routine will do.
```python
from fuzzywuzzy import fuzz
from fuzzywuzzy import process

# Compare sichuan cdi and ccdi titles and add %80+ matches to list.
sichuan_vs_ccdi_title_matches = []
for st in sichuan_titles:
    for ct in ccdi_titles:
        ratio = fuzz.ratio(st,ct)
        if ratio >= 80:
            match = {
                'sichuan_title': st,
                'ccdi_title': ct,
                'match_ratio': ratio,
                }
                sichuan_vs_ccdi_title_matches.append(match)

print('matches: ' + str(len(sichuan_vs_ccdi_title_matches)))

>>> matches: 89
```
Results were pretty good. Examining the string matches I found only 11% were false matches. Here is a representative example of a false match.

* 成都市原市长助理刘俊林涉嫌犯罪被移送司法机关
* 成都市原市长助理周鸿德涉嫌犯罪被移送司法机关
* Former Chengdu Mayorial Assistant Junlin Liu, On Suspition of Criminal Action, Was Handed Over to the Judiciary
* Former Chengdu Mayorial Assistant Hongde Zhou, On Suspition of Criminal Action, Was Handed Over to the Judiciary

These officials hold the same title in the same city. It's likely they are implicated in the same incident. But with Fuzzywuzzy alone this is read as a duplicate. I need to improve accuracy so I don’t miss these valuable reports.

### Stanford Named Entity Recognizer (NER)

Pulling officials’ names from the titles and matching, in addition to Fuzzywuzzy, will improve accuracy. This led me to the NER in [Stanford CoreNLP](https://stanfordnlp.github.io/CoreNLP/) and Lynten Guo’s python wrapper for this tool, [stanfordcorenlp](https://github.com/Lynten/stanford-corenlp).
```python
from stanfordcorenlp import StanfordCoreNLP

# Connect to corenlp server.
nlp = StanfordCoreNLP('http://corenlp.run', lang='zh', port=80)

sichuan_vs_ccdi_title_matches = []
	for match in matches_sc:
		# Get name from first match string.
		sentence = match['sichuan_title']
		ner = nlp.ner(sentence)
		name1 = [item[0] for item in ner if 'PERSON' in item]
		
		# Get name from second match string.
		sentence = match['ccdi_title']
		ner = nlp.ner(sentence)
		name2 = [item[0] for item in ner if 'PERSON' in item]
	
		if name1 == name2:
			match['name'] = name1
			sichuan_vs_ccdi_title_matches.append(match)
		
# Close corenlp server connection.
nlp.close()

print('matches: ' + str(len(verified_matches_sc)))

>>> matches: 76
```

Stanford’s NER is not perfect. It has trouble recognizing unusual names (see example at the bottom) and sometimes only grabs part of the individuals name. However combined with fuzzywuzzy the outcome is highly accurate. There were no false matches and the examination of the raw ccdi data suggests there are around 76 matches. So the risk of false negatives is quite minimal.

If you've made it this far, thank you for reading! If you have any suggestions for improving my code, please shoot me an email at pattersoncharlesl@gmail.com.

### Example of a Name Stanford CoreNLP Can't Handle

* 四川省绵阳高新技术产业开发区党工委书记羊群被调查
* Sichuan Province, Miaoyang National High-tech Industries Development Zone Party Working Committee Secretary Yangqun is Under Investigation

Yangqun in Chinese literally means 'flock of sheep'—an unusal name. Thus Stanford's NER couldn't recognize it. Perhaps Yangqun's parents were keen to remind him of his zodiac.