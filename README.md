# Analysis of voter turnout of Municipalities of Switzerland and some thoughts on causal inference

## Introduction

### Idea
Switzerland consists of 2136 Municipalities as of 1 January 2023.<sup>[[1]](README.md#References)</sup> National elections are held every four years. For each election, the voter turnout is recored by the Federal Statistical Office (FSO). The voter turnout is the proportion of the population entitled to vote that actually voted. Independent of that the Federal Statistical Office regularly collects data on each municipality like population size (residents) or the percentage of foreigners in a Municipality. In this analysis, we take both datasets, merge them and try to find out if there are relations between the voter turnout and other characteristics of the Municipalities. The voter turnout is always considered as the target variable $`Y`$. The other variables are considered as the input $`X_1, X_2, X_3... X_i`$. 

### Prerequisite knowledge

To follow this analysis you should be familiar with following concepts. We will not explain them here since you will find a lot of good teaching material online.

- Mean and standard deviation
- Standardization of variables
- Normal distribution
- Pearson correlation coefficient
- Linear regression with least squares cost function 
- Residuals
- Histogram and Scatter plot

### Recommended readings

For the last two chapters of this article, we use concepts described by Judeal Peral et al. in the following books:

<p align="center">
<img 
   alt="Causal Inference in Statistics: A Primer"
   src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/images/Book-Causal-inference-in-Statistics-A-Primer.jpg" 
   height="250"
/>
<img 
  alt="The Book of Why"
  src="https://github.com/t4d-gmbh/voter-turnout-in-switzerland/blob/main/images/The-Book-of-Why.jpg" 
  height="250"
/>
</p>

### Proceeding

1. Data retrieval
2. Data preprocessing
3. Descriptive analysis
4. Identify correlations and relevant variables
5. Causal inference ( user other title...)
6. Conclusions

## Used tools

The analysis was performed in a *Juypter notebook*. Libraries like Pandas, scipy, matplotlib, seaborn and bokeh were used. You find the jupyter notebook in this repository.

## What we did not do
The focus of this analysis lies on the proceeding itself rather than on the actual outcome. We did not optimize a statistical or a machine learning model to do optimal predictions - since we have already the data of almost all Municipalities there is not much to predict in this case. In return, we tried to find out how we can interpret the discovered correlations in terms of causal relationships, based on the concepts described in the mentioned books.

## Data retrieval

We retrived  the data from following data sources
1. The voter turnouts foe the Federal elections 2023 (opendata.swiss)<sup>[X]</sup>
2. The characteristics of the municipalities (bfs.admin.ch)<sup>[X]</sup>

To ensure the traceability of the analysis, all data can be found in the *data/original* directory of this repository.

## Data preprocessing

You find the preprocessed data files in the direcory *data/preprocessed* These files (CSV) are used for analysis.

### Data cleaning

The first file from openswiss.data also contains the aggregated voter turnout for the cantons and the Confederation. These entries have been deleted and we only use the voter turnouts of the municipalities. Otherwise, it is a valid CSV file that can be imported. The formatting of the second file (xlsx format) has been manually adjusted and exported to CSV. Unknown values that were marked with "X" or "*" in the original file have been replaced with empty values so that they are interpreted as *NaN* values in Python, which is easier to work with in Python.

The second file contained several variables describing the voting behavior with regard to individual political parties. However, these values were not available for numerous municipalities. To simplify the analysis, we deleted these variables and focussed on the more than 30 remaining variables.


### Merging files

The only information we can use to merge the two datasets are the names of the municipalities. As the FSO uses different spellings for some municipalities (e.g. *Zurzach* and *Bad Zurzach*), we have harmonized the names of approximately 30 municipalities. We have also translated the German-language variables into English. The translations are based on the English terms provided by the FSO in the file (*data/original/ts-x-21.03.01-APPENDIX.ods*). Finally the files were merged:
```
data = pd.merge(municipalities, turnouts, on='Municipality', how='inner')
```
### Standization of values


## References

<sup>[1]</sup> [Die institutionellen Gliederungen der Schweiz](https://www.bfs.admin.ch/bfs/de/home/statistiken/querschnittsthemen/raeumliche-analysen/raeumliche-gliederungen/Institutionelle-gliederungen.html)

<sup>[X]</sup> [Federal Statistical Office FSO (opendata.swiss)](https://opendata.swiss/de/dataset/eidg-wahlen-2023/resource/e3e5a96f-171b-4876-9d92-ab7a1dfc8b5f)

<sup>[X]</sup> [Federal Statistical Office FSO: Regionalporträts 2021: Kennzahlen aller Gemeinden](https://www.bfs.admin.ch/bfs/de/home/statistiken/regionalstatistik/regionale-portraets-kennzahlen/gemeinden.assetdetail.15864450.html)

