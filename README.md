---
title: "PropScore"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here is a demonstration of creating analyzing matched data for propensity score analysis using the MatchIt package.  First we create an artifical data set contains a set of coviarates (school size, percentage of minority students, and free and reduced lunch) along with a dependent variable and "treatment" indicator, which we using a public indicating whether or not a student attends a public (1) or private (0) school.  

Running the MatchIt R program is easy.  First we create the model, which is a logisitic regression of the treatment (i.e. public school) regressed upon the following covariates: size, minority, and freelunch.  Next we need to select the data frame to use, which is the propScore data frame that contains all of the variables of interest.  Next we select the ratio, which we set to one indicating that each student in the treatment / public group will be matched with one student in the private / control group.  
```{r}
install.packages("MatchIt")
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
# Ratio means every data point will be matchd with one other data point
m.out = matchit(public ~ size + minority + freeLunch, data = propScore, method = "nearest", ratio = 1)
```
How we are going zelig with the matched data.  Think about the weights they need to be included in the anlaysis
```{r}
summary(m.out)
m.outCSV = match.data(m.out) 
write.csv(m.outCSV, "m.outCSV.csv")
plot(m.out, type = "jitter")
plot(m.out, type = "hist")
```
Analysis
```{r}
z.out = zelig(dep ~ public + minority + freeLunch + size, model = "ls",data = m.out)
summary(z.out)
```

