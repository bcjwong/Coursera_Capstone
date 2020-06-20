# Crime Incidents & Venues Data Analysis in Boston, Massachusetts

(Capstone Project - The Battle of Neighborhoods)

## Introduction/Business Problem
The goal of this capstone project is to explore and extrapolate exisiting data to solve real life problems. Hence, for this project, we will look at which location in Boston is most promising for opening a new restaurant. Specifically, we will explore the crime rates in the 12 districts of Boston, different types of venues in each location, and output the safest and most profitable location to open a restaurant venue. 


## Data acquisition and pre-processing
### Data Sources
	- Crime incident reports will be downloaded from the website *Analyze Boston* (https://data.boston.gov/dataset/crime-incident-reports-august-2015-to-date-source-new-system/resource/12cb3883-56f5-47de-afa5-3b1cf61b257b), which contains numerous open source data sets of Boston. The data set crime incident reports from August 2015 to current date will be used for analysis because of its most up-to-date information. 

	- Due to a lack of district name in the crime incident reports data set, an addition table which has the district name and the district number will be used for mapping the two tables together. This is a fairly straightforward table as there are only 12 districts in Boston.

	-  In order to obtain venue information around the districts in Boston, the Forsquare API will be used to get the most common venues in each district. This will gives us an understanding of the types and number of venues that are located in Boston. 

###Data pre-processing
	-  The *boston_crimerate.csv* file has numerous entries that have NaN parameters. Thus, the first step was to drop all entries that have missing information. Then, the dataset, which has 17 columns (parameters), were simplified for the purpose of this analysis. Impertinent columns were dropped and and the crimerate dataframe was ordered with the most recent date at the top (2019-10-13).

	![df_crime](/images/df_crime.png)
	*figure 1*
	
	-  Note that in figure 1, the data set from the website contains the district code but not the name. This proved to be confusing in the analysis portion. The next step in data pre-processing was to map each district name from the *boston_district.csv* to the processed crime dataframe. The result is shown in figure 2

	![df](/images/df.png)
	*figure 2*

	-  In order to obtain local venues from each district in Boston, the foursqare API was implemented. With each API request (given latitude and longitude coordinates), the design limit was set to 100 venues and a radius of 1200 meters. Here is an example of the venues data set that was returned. 

	![api-preprocess](/images/api-preprocess.png)
	*figure 3*


## Methodology

The data analysis portion can be divided into two sections. The first section explores the crime incident rates in each district in Boston. The latter portion examines the different venues located in each district. Ultimately, these factors will be used to determine the most protifable type of business to open in the safest neighborhood in Boston. 

### Crime incident report methodology
	In this methodology section, 3 main objectives were explored. Which category has the highest number of crime incident reports, which district has the highest number of reported crime incidents, and the top 5 types of crime incidents in each district. 

		1. By grouping the crime data set based on crime incident category, and then sorting the values based on the number of incidents, we are able to obtain the top crime incidents in Boston.

		2. A choropleth map is used to show the number of reported crime incidents in each district. This is achieved by first grouping the dataframe by districts, and then order by number of reports. Then choropleth map is plotted using the folium library. 

		3. We are able to examine the top 5 categories of crime incident reports in each district by utilizing the multiindex object supported by a typical dataframe. By manipulating data and creating hierarchical index, we are able to better understand and analyze the Boston crime dataframe. 

### Venues methodology
	




## Results

## Discussion

## Conclusion

