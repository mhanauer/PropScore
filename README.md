---
title: "PropScore"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here is a demonstration of how to create and analyze matched data for propensity score analysis using the MatchIt package.  First, we create an artificial data set that contains the following set of covariates (school size, percentage of minority students, and free and reduced lunch) along with a dependent variable and "treatment" indicator, indicating whether or not a student attends a public (1) or private (0) school. 

Running the MatchIt R program is easy.  First, we create the model, which is a logistic regression of the treatment (i.e. public school) regressed upon the following covariates: size, minority, and freelunch.  Next, we need to select the data frame to use, which is the propScore data frame that contains all of the variables of interest.  Next, we select the ratio, which we set to one indicating that each student in the treatment / public group will be matched with one student in the private / control group.  
```{r}
library(MatchIt)
set.seed(12345)
studentID = 1:10000
dep = abs(rnorm(10000))
public = c(rep(1,5000), rep(0,5000))
size = abs(rnorm(10000)*10000)
minority = abs(rnorm(10000)/10)
freeLunch = abs(rnorm(10000)/10)
propScore = cbind(public, size, minority, freeLunch)
propScore = as.data.frame(propScore)

m.out = matchit(public ~ size + minority + freeLunch, data = propScore, method = "nearest", ratio = 1)
```
Next, we want to analyze how well the matching procedure worked.  First, we can look at a summary by using the summary function.  The summary function provides pieces of information for the full and the matched data set.  First it provides data on the balance for all of the data without matching.  It provides information the means of the treated (i.e. public as indicated by the 1 for the public variable) and the control (i.e. private as indicated by the 0 for public variable).  It also provides the standard deviation for the control group.  It provides means and sd's for both the treatment and control groups across each of the included covariates.  Next there are QQ columns for the median, mean, and maximum quantiles differences between the treatment and control groups.  Smaller QQ values indicate better matching data.  

The summary of balance for matched data is interpreted in the same way as for the summary of balance for all data with the only difference being that the summary of balance for matched data uses only matched data.  Therefore, the user can compare the differences in means and reductions in quantiles to evaluate if the matching process reduced the observed differences between the public and private group with the matched data.  Finally, there is the percent balance improvement, which provides percentage improvement by using the matched data relative to all the data.  In this example, there is no improvement, because we were able to match all of the data, which is likely because the data were randomly generated.

Finally, there are two plots that researchers can review to evaluate the effectiveness of the matching procedure.  First is the jitter plot, which shows the distribution of unmatched and matched pairs for both treatment and control groups.  You can see how close the data are from the matched and unmatched groups demonstrating the matched groups similarities among the observed covariates.  Finally, we can evaluate a histogram of the matched and raw (i.e. total) data sets to evaluate how much better the matching procedures matched the data.   
```{r}
summary(m.out)
m.outCSV = match.data(m.out) 
write.csv(m.outCSV, "m.outCSV.csv")
plot(m.out, type = "jitter")
plot(m.out, type = "hist")
```
Finally, we can evaluate the impact of being in either a public or private school using the matched data, by evaluating the significance of the public variable.  We can use the Zelig function to create the model that includes covariates.
```{r}
library(Zelig)
z.out = zelig(dep ~ public + minority + freeLunch + size, model = "ls",data = m.outCSV)
summary(z.out)
```

