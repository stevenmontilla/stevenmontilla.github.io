---
title: Blog week 2
layout: default
---
[Return to main page](https://stevenmontilla.github.io)

Prepare a blog post of two paragraphs answering:

Summarize the analytical techniques applied and how the results of those techniques were communicated in text, numbers, tables, or data visualizations
Consider whether you consider this research paper to be reproducibile and whether you consider this paper to be replicable. Refer to the National Academies of Sciences, Engineering and Medicine definitions of reproducibility and replicability and our prior discussions about GIS as a Science.

Wang et al. (2016) developed a methodology to analyse twitter activity during wildfires around the San Diego area in May of 2014. They used the twitter search API to collect tweets that contained the words "wildfire" and "fire" and then a sub-set of tweet containg information about specific locations where fires occured during that time frame. From the totality of tweets collected, they were only interested in those with geospatioal information. They used the 'coordinates' field to filter out unuseful tweets. Having georeferenced tweets, they performed kernel density estimation to understand the spatial pattern of the tweets. This may represent a problem because areas with larger population may produce more twitter activity in general. Thus, the results may appear as a population map which would not provide useful information for the analysis. Therefore, they had to normalize it by making a dual kernel density estimation where the number of tweets in each unit of analysis was divided by its corresponding population value. They also conducted other analysis such as interaction networks or word analysis to understand how people where communicating about the fires.

Regarding replicabicability and reproducibility, 
