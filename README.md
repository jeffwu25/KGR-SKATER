# stgci
Code for implementing kernel graph regression for spatiotemporal causal inference

The main research question being addressed here is: what is the causal effect of the presence of wildfire specific PM 2.5 on the number of respiratory related hospitalizations  each day following significant wildfires in California? 

The general flow of our approach to answer this question is as follows: 

I plan on using real hospitalization data from the HCAI to estimate counterfactual outcomes i.e. how many hospitalizations would have been observed if no wildfire had occurred in a given area, based on the observed data from other areas. I intend to track the subsequent number of hospitalizations following several big wildfires over the course of the 2010s in California. In order to impute missing counterfactual outcomes (what hospitalization counts would have been observed had no wildfire smoke had been present in a given zipcode), we will be applying a novel spatiotemporal modeling framework. We will define a partially observed graph with nodes representing each zipcode of California. First, we will learn the structure of the graph. Then we will use that learnt structure to estimate the underlying signal of the graph. Finally, we will use that signal to estimate values for unobserved nodes of the graph i.e. predicting counterfactual values that we were not able to observe. 

--------------------------------------------------------------------------------------------------------------------------------------------------------------

Each of the code files in this repository represent a step in the overall construction of our graph regression model. The file that contains the full analysis AKA the main file of interest is SoA County Data Analysis.Rmd . It is good to run this file first before experimenting with any of the secondary files as there may be some dependencies. 

Currently, we only working on our analysis method via toy data. That is, county data from the SoA year to year; the hospitalizations are synthetically generated from a LGCP. Our goal is to eventually examine daily respiratory related hospitalizations following significant wildfires between 2010-2019 at the zipcode level across California.

1. Covariance Matrix Filling Functions.Rmd

Implements theorems and corollaries from paper, "Explicit solutions to correlation matrix completion problems, with an application to risk management and insurance" by Georgescu et al. (2018), that identifies certain closed form solutions to updating a given covariance matrix's values while simultaneously inducing sparseness (0s) in the corresponding entries in the precision matrix (inverse of the given covariance matrix) 

2. EDA on subindices.Rmd

Conducts some basic exploratory data analysis on the subindices (11 of them) that make up the social deprivation score provided by the Society of Actuaries (SoA) e.g. boxplots, checking for normality, time seres visualizations

3. HUGE model selection.Rmd

Experiments with different aspects of the HUGE (high-dimensional undirected graph estimation) package in R. There are several options for the method of estimating  the graph and also for selecting the optimal graph based on different criterion. Sine our toy data only comprised of a time series of length 10, we decided to perform two different resampling techniques (standard bootstrap and Gaussian process interpolation) on the data so that there were more replications/observations for HUGE to learn the graph on. Then, we decided to perform a hypothesis test proposed by Cai et al. (2012) called, "Two-Sample Covariance Matrix Testing and Support Recovery in High-Dimensional and Sparse Settings", designed to test the equality of two covariance matrices to see how consistent the estimated grpah was across different bootstrap methods, estimation methods, and model selction crtierion. 

4. Ideal Cluster Number.Rmd

Explores how many clusters might be optimal for SKATER to cluster the counties of Califoria into. The two R functions we used for evaluating the ideal number of clusters were: fviz_nbclust() and silhouette(). These functions perform the same procedure of calculating the average width for different number of clusters but in slightly different ways. We used these two functions to compare clustering results across different subsets of SoA subindices in a stepwise manner to get an idea of how many clusters we want SKATER to create later on. 

5. Inducing Sparseness Exercise.Rmd

My  initial attempt at inducing sparseness into the precision matrix before I created the Covariance Matrix Filling Functions.Rmd file. This approach is not correct in comparison to the one presented in the paper by Georgescu et al. because it does not take into account the condition of positive definiteness for the covariance matrix. 

6. SKATER validation.Rmd

Confirms that the SKATER algorithm is correctly grouping units together based on associations between units represented in the data's covariance matrix. This is done by artifically inducing sparseness to a full covariance matrix (calculated from our toy data) and then simulating new data with that covariance matrix from the multivariate normal distribution so that we have a new toy dataset. One would think that performing a SKATER clustering on this new data (which was generated with a covariance matrix that has a clustered pattern) should be able to correctly separate the units into clusters that agree with the pattern of sparseness we choose to induce. This is a work in progress...



