# stgci
Code for implementing kernel graph regression for spatiotemporal causal inference

The main research question being addressed here is: what is the causal effect of the presence of wildfire specific PM 2.5 on the number of respiratory related hospitalizations  each day following significant wildfires in California? 

The general flow of our approach to answer this question is as follows: 

Each of the code files in this repository represent a step in the overall construction of our graph regression model. The file that contains the full analysis AKA the main file of interest is SoA County Data Analysis.Rmd . It is good to run this file first before experimenting with any of the secondary files as there may be some dependencies. 

Currently, we only working on our analysis method via toy data. That is, county data from the SoA year to year; the hospitalizations are synthetically generated from a LGCP. Our goal is to eventually examine daily respiratory related hospitalizations following significant wildfires between 2010-2019 at the zipcode level across California.
