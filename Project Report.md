# Crime Incidents & Venues Data Analysis in Boston
Coursera - Applied Data Science Capstone  
Capstone Project - The Battle of Neighborhoods

## Introduction/Business Problem
The goal of this capstone project is to explore and extrapolate exisiting data to solve real life problems. Hence, for this project, we will look at which location in Boston is most promising for opening a new restaurant. Specifically, we will explore the crime rates in the 12 districts of Boston, different types of venues in each location, and output the safest and most profitable location to open a restaurant venue. 


## Data acquisition and Pre-processing
### Data Sources
- Crime incident reports will be downloaded from the website *Analyze Boston* (https://data.boston.gov/dataset/crime-incident-reports-august-2015-to-date-source-new-system/resource/12cb3883-56f5-47de-afa5-3b1cf61b257b), which contains numerous open source data sets of Boston. The data set crime incident reports from August 2015 to current date will be used for analysis because of its most up-to-date information. 

- Due to a lack of district name in the crime incident reports data set, an addition table which has the district name and the district number will be used for mapping the two tables together. This is a fairly straightforward table as there are only 12 districts in Boston.

-  In order to obtain venue information around the districts in Boston, the Forsquare API will be used to get the most common venues in each district. This will gives us an understanding of the types and number of venues that are located in Boston. 

- Boston geospatial map downloaded from *Analyze Boston* (https://data.boston.gov/dataset/boston-neighborhoods/resource/13ee2b65-6547-4168-b112-83995f138602). 

### Data Pre-processing
-  The *boston_crimerate.csv* file has numerous entries that have NaN parameters. Thus, the first step was to drop all entries that have missing information. Then, the dataset, which has 17 columns (parameters), were simplified for the purpose of this analysis. Impertinent columns were dropped and and the crimerate dataframe was ordered with the most recent date at the top (2019-10-13).

<p align="center">
  <img src="/images/df_crime.png"> <br> Figure 1
</p>
	
-  Note that in figure 1, the data set from the website contains the district code but not the name. This proved to be confusing in the analysis portion. The next step in data pre-processing was to map each district name from the *boston_district.csv* to the processed crime dataframe. The result is shown in figure 2

	![df](/images/df.png)  
								*Figure 2*

-  In order to obtain local venues from each district in Boston, the foursqare API was implemented. With each API request (given latitude and longitude coordinates), the design limit was set to 100 venues and a radius of 1200 meters. Here is an example of the venues data set that was returned. 

	![api-preprocess](/images/api-preprocess.png)  
								*Figure 3*


## Methodology

The data analysis portion can be divided into two sections. The first section explores the crime incident rates in each district in Boston. The latter portion examines the different venues located in each district. Ultimately, these factors will be used to determine the most protifable type of business to open in the safest neighborhood in Boston. 

### Crime Incident Report Methodology
In this methodology section, 3 main objectives were explored. Which category has the highest number of crime incident reports, which district has the highest number of reported crime incidents, and the top 5 types of crime incidents in each district. 

1. By grouping the crime data set based on crime incident category, and then sorting the values based on the number of incidents, we are able to obtain the top crime incidents in Boston.

	```
	district_crime = df.groupby('DISTRICT').count()
	district_crime.sort_values(by='CATEGORY', ascending=False, inplace=True)
	```

2. A choropleth map is used to show the number of reported crime incidents in each district. This is achieved by first grouping the dataframe by districts, and then order by number of reports. Then choropleth map is plotted using the folium library. 

	```
	bos_map.choropleth(
    geo_data=bos_geo,
    data=district_crime,
    columns=['DISTRICT', 'CATEGORY'],
    key_on='feature.properties.DISTRICT',
    fill_color='YlOrRd', 
    fill_opacity='0.6', 
    line_opacity='0.2',
    legend_name='Crime Incident in Boston'
    )
    ```

3. We are able to examine the top 5 categories of crime incident reports in each district by utilizing the multiindex object supported by a typical dataframe. By manipulating data and creating hierarchical index, we are able to better understand and analyze the Boston crime dataframe. 
	
	```
	t1 = df.groupby(['NAME', 'CATEGORY']).count()  
	for idx, df_sel in t1.groupby(level=[0]):
    y = df_sel.sort_values(by=['COUNT'], ascending=False)
    head = y.head()
    multiindex.append(head)
    ```


### Venues Explorartion & Clustering
1. The first step was to clean the dataframe of unwanted parameters and obtain required credientials for API calls. 

2. Exploring neighborhoods in Boston requires making API requests to Foursquare. The function that automates the request of calling for each district is given in the code snippet below. The number of venues returned is limited to 100 and the area searched around the given GPS coordinates is set to 1200 meters. 
	```
	bos_venues = getNearbyVenues(names=bos_df['NAME'],
                                   latitudes=bos_df['LAT'],
                                   longitudes=bos_df['LONG']
                                  )
    ``` 
By calling the groupby function, we are able to see the the number of venues in each neighborhood. 

	![explore_venues](/images/explore_venues.png)  
								*Figure 4*

	![unique_cat](/images/unique_cat.png)  
								*Figure 4*

3. In order to analyze each neighborhood in the data set, the data needed to be converted into one-hot encoding format. The point of converting data set is because we are examining categorical data; The different types of venues that are located in each neighborhood. Many machine learning algorithms, such as K-means clustering used in this project, cannot perform on raw data that is returned by the API. Therefore, we need to convert the data into numerical form.  

After performing one-hot encoding on the dataset, the dataframe is sorted to output the 10 most common venues in each neighborhood. 
	
 	![unique_cat](/images/unique_cat.png)  
								*Figure 4*

4. Becuase there are similar venues in each neighborhood, I decided to implement K-means clustering to group the neighborhoods. K-means algorithm is one of the most popular unsupervised machine learning algorithm. I set the number of clusters to 4 for this project. 
```
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(bos_grouped_clustering)
```
*NOTE: I will explain some tests I performed regarding K-means as well as some thoughts about this implementation at the end of this report.*

5. Visualization and cluster examination
After clustering the data points based on venues, a map of Boston with cluster points is created for visualization. Each cluster with district and top venues can also be printed out for more information about the types of venues that reside in the districts. 


## Results
### Crime Incident Report Results
Now that we have the grouped the crime incidents based on the number of reports, we can use a bar graph to see what the top 10 crime incidents are bewteen Janurary 2019 and October 2019.

![top10_barh](/images/top10_barh.png)  
 								*Figure 8*

Let's also take a look at the choropleth map of districts in Boston based on the number of crime incident reports. 
	
![choropleth](/images/choropleth.png)  
 								*Figure 9*

Lastly, we can see the top 5 crime incident category for each district. Due to space limitations, only 3 of the districts will be shown as an example.

![district_top5](/images/district_top5.png)  
 								*Figure 8*

### Venues Explorartion & Clustering Results
After performing K-means clustering on the dataset grouped based on district and venues, we can output a map that shows the districts based on cluster results. The dots represent each district's approximate location and the colors represent in which cluster they belong in. We are able to see that the dots are clustered into 4 groups, the number that we specified. 

 	![cluster_map](/images/cluster_map.png)  
  	*Figure 8*

In addition, we can examine each cluster and see which districts are together. 

Cluser 1:
 	![c0](/images/c0.png)  
  								*Figure 8*

Cluser 2:
 	![c1](/images/c1.png)  
  								*Figure 8*

Cluser 3:
 	![c2](/images/c2.png)  
  								*Figure 8*

Cluser 4:
 	![c3](/images/c3.png)  
  								*Figure 8*


## Discussion
### Crime Incident Report Discussion






### Venues Exploration & Clustering Discussion




## Conclusion

