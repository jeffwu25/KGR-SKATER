# stgci
Code for implementing spatially clustered kernel graph regression with INLA for high dimensional data (with causal inference extensions)

The main research question being addressed here is: What is the effect of wildfires on respiratory related comorbidities each day following significant wildfires in California? 

To answer this question, we will be applying a novel spatiotemporal modeling framework that incorporates complex dependence structures into a model separately in a parsimonious and interpretable fashion. First, we will define a partially observed graph with nodes representing each zipcode of California and learn its structure. Then, we will use that learned structure to estimate the underlying signal of the graph. Finally, we will use that signal to estimate values for unobserved nodes of the graph i.e. predicting counterfactual values that we were not able to observe. 

Project 1: In the first project, we simply wanted to establish a procedure for building a model and making predictions at each county in California that is easy to implement and computationally efficient. While we tried to request the data needed for project 2 from the HCAI, we chose to use a dataset on respiratory related mortality from 2014-2019 from Cal-ViData as our response variable for project 1. 

Project 2: In the second project, we wanted to apply our procedure in a causal inference setting. The predictions that our model makes, based on the observed data from regions that did not experience a wildfire, will be for "missing" counterfactual outcomes (respiratory related hospitalizations from the HCAI) i.e. outcomes if no wildfire had occurred in a given area. This way, we will be able to make a comparison in order to ascertain the causal effect of wildfires on respiratory related comorbidities. 

--------------------------------------------------------------------------------------------------------------------------------------------------------------

The file to start with is SoA County Data Analysis.Rmd. It is good to run this file first before experimenting with any of the secondary files as there may be some dependencies as it contains the first steps of the analysis. THE FILES IN THE SUPPLEMENTARY FOLDER ARE NOT NECESSARY TO RUN IN ORDER TO EXECUTE OUR MODELING PROCEDURE??? 

1. Covariance Matrix Filling Functions.Rmd

Implements theorems and corollaries from the paper, "Explicit solutions to correlation matrix completion problems, with an application to risk management and insurance" by Georgescu et al. (2018), that identifies certain closed form solutions to updating a given covariance matrix's values while simultaneously inducing sparseness (0s) in the corresponding entries in the precision matrix (inverse of the given covariance matrix) 

2. EDA on subindices.Rmd

Conducts some basic exploratory data analysis on the subindices (11 of them) that make up the social deprivation score provided by the Society of Actuaries (SoA) e.g. boxplots, checking for normality, time series visualizations

3. EPA Data Download.Rmd

Downloads, cleans, and aggregates data sourced from the US EPA air quality system API, with the end goal of calculating a kernel gram matrix K which relates measurements of different pollutants together across time. This involves identifying a good set of EPA stations that exhibit a good spatial coverage of air quality levels across California. Certain stations only measure certain pollutants, certain counties do not have measurements for all counties, and certain stations are missing data, which made this more complicated. Once, the data is obtained from this set of stations, it needs to be aggregated from the daily to monthly level and put into a single dataframe, which makes calculating K relatively straightforward. 

5. HUGE model selection.Rmd

Experiments with different aspects of the HUGE (high-dimensional undirected graph estimation) package in R. There are several options for the method of estimating  the graph and also for selecting the optimal graph based on different criteria. Since our toy data only comprised of a time series of length 10, we decided to perform two different resampling techniques (standard bootstrap and Gaussian process interpolation) on the data so that there were more replications/observations for HUGE to learn the graph on. Then, we decided to perform a hypothesis test proposed by Cai et al. (2012) called, "Two-Sample Covariance Matrix Testing and Support Recovery in High-Dimensional and Sparse Settings", designed to test the equality of two covariance matrices to see how consistent the estimated graph was across different bootstrap methods, estimation methods, and model selection criterion. 

5. Ideal Cluster Number.Rmd

Explores how many clusters might be optimal for SKATER to cluster the counties of California into. The two R functions we used for evaluating the ideal number of clusters were: fviz_nbclust() and silhouette(). These functions perform the same procedure of calculating the average width for different numbers of clusters but in slightly different ways. We used these two functions to compare clustering results across different subsets of SoA subindices in a stepwise manner to get an idea of how many clusters we want SKATER to create later on. 

6. Important Plots.Rmd

Creates various plots (mainly heatmaps) that will be used in presentations, papers, etc. 

8. Mortality Analysis.Rmd

Downloads "RespiratoryMortality1423.xlsx" dataset and explores the patterns in respiratory related mortality across age groups, counties, and years across California. Primarily, we want to see the frequency of 0 or "< 11" values in the total death columns because this affects whether we use a standard Poisson or zero-inflated Poisson model for our response and whether we should apply a truncation to that distribution because "< 11" is not easy to work with. We ended up deciding to impute all of the "< 11" values in order to avoid using a truncated distribution. This imputation was carried out in a separate file (see Mortality EM Algorithm AND ... files) 

9. Mortality EM Algorithm.Rmd

Imputes "< 11" values from the "RespiratoryMortality1423.xlsx" dataset via the EM algorithm (both a standard Poisson and zero-inflated Poisson version). This algorithm ensures that the maximum likelihood estimate for lambda is used to predict the missing value. 

11. Poisson-LRT.Rmd

Implements a likelihood ratio test (basically a hypothesis test) to see whether a standard Poisson and zero-inflated Poisson distribution is more appropriate for our mortality dataset. (Work in progress) 

13. SKATER validation.Rmd

Confirms that the SKATER algorithm is correctly grouping units together based on associations between units represented in the data's covariance matrix. This is done by artificially inducing sparseness to a full covariance matrix (calculated from our toy data) and then simulating new data with that covariance matrix from the multivariate normal distribution so that we have a new toy dataset. One would think that performing a SKATER clustering on this new data (which was generated with a covariance matrix that has a clustered pattern) should be able to correctly separate the units into clusters that agree with the pattern of sparseness we choose to induce.

12. SoA County Data Analysis

Downloads "SoA.data.1019.xlsx" dataset and performs the first two steps of our proposed procedure. First, spatially contiguous clusters are generated using SKATER by pruning a minimum spanning tree. Then, the deprivation score data from the SoA is aggregated into their respective clusters via a population weighted mean and fed into the HUGE function, which estimates the optimal graph structure i.e. which edges are present between the nodes (which represent the clusters). The HUGE function has several options for estimation method and choice of regularization which can result in different resulting graphs. Finally, we take the adjacency matrix from our HUGE output and apply a graph filter transformation to obtain our graph filter H (see "GLS Kernel Regression for Network-Structured Data" by Antonian et al. 2021) for the equations. 


