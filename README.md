# Analysis of voter turnout of Municipalities of Switzerland and some thoughts on causal inference

## Introduction

### Idea
Switzerland consists of 2136 Municipalities as of 1 January 2023. National elections are held every four years. For each election, the voter turnout is recored by the Federal Statistical Office (FSO). The voter turnout is the proportion of the population entitled to vote that actually voted. Usually, the turnout is around 50% and varies between 20% and 80% among the municipalities. Independent of that the Federal Statistical Office regularly collects data on each municipality like population size or the percentage of foreigners in a Municipality.

In this analysis, we take both datasets, merge them and try to find out if there are relations between the voter turnout and other characteristics of the Municipalities. The voter turnout is always considered as the target variable y. The other variables are considered as the input variables x_1, x_2, x_3... x_i. 
