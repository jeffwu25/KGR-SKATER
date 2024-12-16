# KGR-SKATER
Code for implementing spatially clustered kernel graph regression with INLA for high dimensional data (which we hope to extend to spatiotemporal causal inference)

In this project, we develop a novel spatiotemporal modeling framework that incorporates complex dependence structures into a model separately in a parsimonious and interpretable fashion. Then, we demonstrate this framework's utility with an application and simulation study. 

The overarching research question for this project is: can we model respiratory related deaths in California using spatial variation in social deprivation score and temporal variation in air pollutant levels? 

--------------------------------------------------------------------------------------------------------------------------------------------------------------

## Data folder: 

This folder contains all the datasets and geographic files needed to run the analysis from project 1. The shapefiles are needed to create a spatial data frame (SPDF) for the SKATER step. The Cal-ViDa dataset contains our response variable: respiratory related mortality for each county. The SoA dataset contains social deprivation score data for each county. Finally, the EPA folder contains air quality data that I queried from the EPA's website. Downloading the aggregated folder which has the daily data from 7 different pollutants combined and aggregated into one dataset for each year (at the county and monthly level) should be sufficient to carry through the steps of the analysis; otherwise, one can just download the final aggregated dataset which is just one singular dataframe. If all else fails, one can download Workspace9.11 which should have all the objects necessary to run the analysis as well. 

## Exploratory Analysis folder: 

1. Covariance-Matrix-Filling-Functions.Rmd 

Implements theorems and corollaries from the paper, "Explicit solutions to correlation matrix completion problems, with an application to risk management and insurance" by Georgescu et al. (2018), that identifies certain closed form solutions to updating a given covariance matrix's values while simultaneously inducing sparseness (0s) in the corresponding entries in the precision matrix (inverse of the given covariance matrix) 

2. EDA-on-subindices.Rmd  

Conducts some basic exploratory data analysis on the subindices (11 of them) that make up the social deprivation score provided by the Society of Actuaries (SoA) e.g. boxplots, checking for normality, time series visualizations

3. HUGE-model-simulations-1.Rmd

Experiments with different aspects of the HUGE (high-dimensional undirected graph estimation) package in R. There are several options for the method of estimating  the graph and also for selecting the optimal graph based on different criteria. Since our toy data only comprised of a time series of length 10, we decided to perform two different resampling techniques (standard bootstrap and Gaussian process interpolation) on the data so that there were more replications/observations for HUGE to learn the graph on. Then, we decided to perform a hypothesis test proposed by Cai et al. (2012) called, "Two-Sample Covariance Matrix Testing and Support Recovery in High-Dimensional and Sparse Settings", designed to test the equality of two covariance matrices to see how consistent the estimated graph was across different bootstrap methods, estimation methods, and model selection criterion. 

4. HUGE-model-simulations-2.Rmd

More experiments with the HUGE package. The main objective of this file is to determine whether or not estimating a graph with edges is appropriate for this project. First, partial correlations are calculated between the residuals of various linear regression models. Next, a series of experiments are conducted to understand the sensitivity of the estimated graphs produced by toggling with each of the inputs in the huge() and huge.select() functions e.g. tuning parameters lambda (in huge), gamma (for EBIC in huge.select) or stars.thresh (for STARS in huge.select) 

5. Ideal-Cluster-Number.Rmd

Explores how many clusters might be optimal for SKATER to cluster the counties of California into. The two R functions we used for evaluating the ideal number of clusters were: fviz_nbclust() and silhouette(). These functions perform the same procedure of calculating the average width for different numbers of clusters but in slightly different ways. We used these two functions to compare clustering results across different subsets of SoA subindices in a stepwise manner to get an idea of how many clusters we want SKATER to create later on. 

6. Important-Plots.Rmd

Creates various plots (mainly heatmaps) that will be used in presentations, papers, etc. 

7. Mortality-EM-Algorithm.Rmd

Imputes "< 11" values from the "respmortality1423.csv" dataset via the EM algorithm (both a standard Poisson and zero-inflated Poisson version). This algorithm ensures that the maximum likelihood estimate for lambda is used to predict the missing value. 

8. Poisson-LRT-v2.Rmd 

Implements a likelihood ratio test to see whether a standard Poisson and zero-inflated Poisson distribution is more appropriate for our mortality dataset. This is basically a hypothesis test to see whether going from one model to another increases the log likelihood by a statistically significant amount. 

9. SKATERvalidation.Rmd

Confirms that the SKATER algorithm is correctly grouping units together based on associations between units represented in the data's covariance matrix. This is done by artificially inducing sparseness to a full covariance matrix (calculated from our toy data) and then simulating new data with that covariance matrix from the multivariate normal distribution so that we have a new toy dataset. One would think that performing a SKATER clustering on this new data (which was generated with a covariance matrix that has a clustered pattern) should be able to correctly separate the units into clusters that agree with the pattern of sparseness we choose to induce.

10. Synthetic-INLA.Rmd

Tests the implementation of various INLA models which will be included as reference models in project 1. We fit a Poisson GLMM, a Besag-York-Mollie model, and our proposed kernel graph regression model to data simulated from a Log Gaussian Cox Process to make sure that the model is being specified and working correctly going into the actual application study. Additionally, the Poisson GLMM model is fit on two more synthetic datasets: (1) a Poisson process with a fixed intensity and a seasonal pattern and (2) a Poisson process with a deterministic intensity (linearly increasing) and a seasonal pattern.  

## Main Analysis folder: 

1. EPA-Data-Download-v6.Rmd (latest two versions included) 

Downloads, cleans, and aggregates data sourced from the US EPA air quality system API, with the end goal of calculating a kernel gram matrix K which relates measurements of different pollutants together across time. This involves identifying a good set of EPA stations that exhibit a good spatial coverage of air quality levels across California. Certain stations only measure certain pollutants, certain counties do not have measurements for all counties, and certain stations are missing data, which made this more complicated. Once, the data is obtained from this set of stations, it needs to be aggregated from the daily to monthly level, from the county to the cluster level, and then put into a single dataframe. This makes calculating K relatively straightforward. 

2. INLA-Mortality-8.1.Rmd (and 8.2)

Synthesizes all the individual steps of our proposed procedure into a single markdown. Most of the code is from the other files in this main analysis folder and some from the exploratory analysis folder. However, this file has more supplementary text and figures than the others which should make it more informative to go through top to bottom for those interested in understanding the high level steps of the analysis performed for this project. The finer steps and details of our procedure are only included in the other files in this folder. The knitted versions of these markdowns are included as well. Note, the number following INLA-Mortality e.g. 8.1 denotes that 8 clusters were used in the SKATER clustering step and that setting 1 was used (unconstrained). This number is important because we consider many numbers of clusters (from 5 to 10) and each of the 3 settings for each of those numbers as a separate fitted model. Hence, there are 18 different models which we fit and evaluate the performance of in the end.  

3. Main-Mortality-Analysis-v3.Rmd (latest two versions included) 

Downloads "RespiratoryMortality1423.xlsx" dataset and explores the patterns in respiratory related mortality across age groups, counties, and years across California. Primarily, we want to see the frequency of 0 or "< 11" values in the total death columns because this affects whether we use a standard Poisson or zero-inflated Poisson model for our response and whether we should apply a truncation to that distribution because "< 11" is not easy to work with. We ended up deciding to impute all of the "< 11" values in order to avoid using a truncated distribution. This imputation was carried out in a separate file (see Mortality EM Algorithm AND ... files) 

4. SoA-Data-Analysis-v3.Rmd (latest two versions included)

Downloads "SoA.data.1019.xlsx" dataset and performs the first two steps of our proposed procedure. First, spatially contiguous clusters are generated using SKATER by pruning a minimum spanning tree. Then, the deprivation score data from the SoA is aggregated into their respective clusters via a population weighted mean and fed into the HUGE function, which estimates the optimal graph structure i.e. which edges are present between the nodes (which represent the clusters). The HUGE function has several options for estimation method and choice of regularization which can result in different resulting graphs. Finally, we take the adjacency matrix from our HUGE output and apply a graph filter transformation to obtain our graph filter H (see "GLS Kernel Regression for Network-Structured Data" by Antonian et al. 2021) for the equations. 

