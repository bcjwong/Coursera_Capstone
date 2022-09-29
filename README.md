# Crime Incidents & Venues Data Analysis in Boston
Coursera - Applied Data Science Capstone  
Capstone Project - The Battle of Neighborhoods  
Benjamin Wong

## Introduction/Business Problem
The goal of this capstone project is to explore and extrapolate exisiting data to solve real life problems. Hence, for this project, we will look at which location in Boston is most promising for opening a new restaurant. Specifically, we will explore the crime rates in the 12 districts of Boston, different types of venues in each location, and output the safest and most profitable location to open a restaurant venue. 


## Data acquisition and Pre-processing
### Data Sources
- Crime incident reports will be downloaded from the website *Analyze Boston* (https://data.boston.gov/dataset/crime-incident-reports-august-2015-to-date-source-new-system/resource/12cb3883-56f5-47de-afa5-3b1cf61b257b), which contains numerous open source data sets of Boston. The data set crime incident reports from August 2015 to current date will be used for analysis because of its most up-to-date information. 

- Due to a lack of district name in the crime incident reports data set, an additional table which has the district name and the district number will be used for mapping the two tables together. This is a fairly straightforward table as there are only 12 districts in Boston.

-  In order to obtain venue information around districts in Boston, the Forsquare API will be used to get the most common venues in each district. This will gives us an understanding of the types and number of venues that are located in Boston. 

- Boston geospatial map downloaded from *Analyze Boston* (https://data.boston.gov/dataset/boston-neighborhoods/resource/13ee2b65-6547-4168-b112-83995f138602). 

### Data Pre-processing
-  The *boston_crimerate.csv* file has numerous entries that have NaN parameters. Thus, the first step was to drop all entries that have missing information. Then, the data set, which has 17 columns (parameters), was simplified for the purpose of this analysis. Impertinent columns were dropped and and the crimerate dataframe was ordered with the most recent date at the top (2019-10-13).

<p align="center">
  <img src="/images/df_crime.png"> <br> Figure 1
</p>
	
-  Note that in figure 1, the data set from the website contains the district code but not the name. This proved to be confusing in the analysis portion. The next step in data pre-processing was to map each district name from the *boston_district.csv* to the processed crime dataframe. The result is shown in figure 2.

<p align="center">
  <img src="/images/df.png"> <br> Figure 2
</p>

-  In order to obtain local venues from each district in Boston, the foursqare API was implemented. For each API request (given latitude and longitude coordinates), the return limit was set to 100 venues and a radius of 1200 meters. Here is an example of the venues data set that was returned. 

<p align="center">
  <img src="/images/api-preprocess.png"> <br> Figure 3
</p>


## Methodology

The data analysis portion can be divided into two sections. The first section explores the crime incident rates in each district in Boston. The latter portion examines the different venues located in each district. Ultimately, these factors will be used to determine the most protifable type of business to open in the safest neighborhood in Boston. 

### Crime Incident Report Methodology
In this methodology section, 3 main objectives were explored. Which category has the highest number of crime incident reports, which district has the highest number of reported crime incidents, and the top 5 types of crime incidents in each district. 

1. By grouping the crime data set based on crime incident category, and then sorting the values based on the number of incidents, we are able to obtain the top crime incidents in Boston.

	```
	crime_category = df.groupby('CATEGORY').count()
	crime_top10 = crime_category['DISTRICT'].head(10)
	```

2. A choropleth map is used to show the number of reported crime incidents in each district. This is achieved by first grouping the dataframe by districts, and then ordering by number of entries. The choropleth map is plotted using the folium library. 

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

2. Exploring neighborhoods in Boston requires making API requests to Foursquare. The function that automates the request of calling for each district is given in the code snippet below. The number of venues returned is limited to 100 and the area searched around the given GPS coordinates is set to 1500 meters. 
	```
	bos_venues = getNearbyVenues(names=bos_df['NAME'],
                                   latitudes=bos_df['LAT'],
                                   longitudes=bos_df['LONG']
                                  )
    ``` 
<p align="center">
  <img src="/images/explore_venues.png"> <br> Figure 4
</p>

By calling the groupby function, we are able to see the the number of venues in each district. 

<p align="center">
  <img src="/images/unique_cat.png"> <br> Figure 5
</p>

3. In order to analyze each neighborhood in the data set, the data needed to be converted into one-hot encoding format. The point of converting the data set is because we are examining categorical data; The different types of venues that are located in each neighborhood. Many machine learning algorithms, such as K-means clustering used in this project, cannot perform on raw data that is returned by the API. Therefore, we need to convert the data into numerical form.  

<p align="center">
  <img src="/images/onehot.png"> <br> Figure 6
</p>

After performing one-hot encoding on the dataset, the dataframe is sorted to output the 10 most common venues in each neighborhood. 

<p align="center">
  <img src="/images/district_venues.png"> <br> Figure 7
</p>


4. Becuase there are similar venues in each neighborhood, I decided to implement K-means clustering to group the districts. K-means algorithm is one of the most popular unsupervised machine learning algorithm. For this project, the number of clusters is set to 4. 
```
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(bos_grouped_clustering)
```
*NOTE: I will explain some tests I performed regarding K-means as well as some thoughts about this implementation at the end of this report.*

5. The last part is visualization and cluster examination. After clustering the data points based on venues, a map of Boston with cluster points is created for visualization. Each cluster can be printed out for more information about the types of venues that reside in the clustered districts. 


## Results
### Crime Incident Report Results
Now that we have the grouped the crime incidents based on the number of reports, we can use a bar graph to see the top 10 crime incidents bewteen Janurary 2019 and October 2019.

<p align="center">
  <img src="/images/top10_barh.png"> <br> Figure 8
</p>

We can conclude that motor vehicle accident reponse is the most prominent recorded incident between Janurary 2019 and October 2019. The second most frequent is medical assistance incident reports. The most frequent "criminal" incident is larceny, which has 5,673 recorded cases.  


Let's also take a look at the choropleth map of districts in Boston based on the number of crime incident reports. 
	
<p align="center">
  <img src="/images/choropleth.png"> <br> Figure 9
</p>

<!-- ![choropleth](/images/choropleth.png)  
 								*Figure 9* -->
From the choropleth map, we can see that the districts Roxbury and Dorchester have the highest number of recorded crime incidents. Districts such as Downtown/Charlestown, East Boston, and West Roxbury have the least number of recorded incidents.

Lastly, we can see the top 5 crime incident category for each district. Due to space limitations, only 3 of the districts will be shown as an example.

<p align="center">
  <img src="/images/district_top5.png"> <br> Figure 10
</p>

From figure 10, we are able to see that almost all of the districts have motor vehicle accidents and medical assitance as the most frequently reported crime incident.  

### Venues Explorartion & Clustering Results
After performing K-means clustering on the dataset grouped based on district and venues, we can output a map that shows the districts based on cluster results. The dots represent each district's approximate location and the colors represent in which cluster they belong in. We are able to see that the dots are clustered into 4 groups, the number that we specified. 

<p align="center">
  <img src="/images/cluster_map.png"> <br> Figure 11
</p>

In addition, we can examine each cluster and see which districts are grouped together. 

<p align="center">
  <img src="/images/c0.png"> <br> Figure 12: Cluster 1
</p>

<p align="center">
  <img src="/images/c1.png"> <br> Figure 13: Cluster 2
</p>

<p align="center">
  <img src="/images/c2.png"> <br> Figure 14: Cluster 3
</p>

<p align="center">
  <img src="/images/c3.png"> <br> Figure 15: Cluster 4
</p>


## Discussion
### Crime Incident Report Discussion
As seen in the results section, we can conclude that Boston is a relatively safe place to start a business. Of course, we will need to compare this data with data from other cities. But from what we can see, most crime incident reports in most districts are related to motor vehicle accident responses and medical assistance. These are less severe crimes. In addition, it appears that areas such as Roxbury, South End, and Dorchester have the highest frequency of crime incident reports. Districts such as Downtown/Charlestown, West Roxbury, and East Boston have the least number of reports. 

Thus, considering the safety of the restaurant/venue, it is recommended to open in districts that have less crime incident reports. Downtown/Charlestown, West Roxbury, and East Boston should be highly considered as optimal locations over other areas. 

### Venues Explorartion & Clustering Discussion
After clustering, we can see that the Downtown/Charlestown area is clustered into purple dots (cluster 2). After closer examination, we see that the districts in cluser 2 all have Italian restaurants, parks and food related venues. Cluster 1 includes districts that have many food related venues in general. Cluster 3 is clustered due to its high prominence of Caribbean restaurants. Cluster 4 is clustered due to its number of stores and cafes. 

Thus, if one wants to open an Italian restaurant, it is recommended to avoid opening in Downtown/Charlestown, East Boston, South Boston, South End areas. There will undoubtedly be competition. This is just an example and can be applied to other scenarios. The entrepreneurial individual can decide which district has the least competition for his/her venue business based on already established venues. 


## Conclusion
Two of the most important factors to look at when one decides to open a venue business is the safety of the location and local area competition. This project dives into examining the crime incident reports in each district in Boston as well as local venues in each district. When one is opening a restaurant, he or she should pick the district with minimal criminal incident reports and least local venue competition for his or her business. 


## Future Directions and Project Retrospection
This is only the first step to deciding the optimal location to open a restaurant revenue. In order to make a more educated decision, more data sets and better processed parameters are needed. Future steps include obtaining more data about each district, such as district economy and income. Furthermore, the data set is only dated during the year 2019 for simplicity. More historical and up-to-date data is needed for more accurate analysis. 

In retrospect, K-means may not have been the most optimal machine learning algorithm for this project. I tried computing the best K value using the elbow method, but the result was not as straight forward (I included the elbow method code in the jupyter notebook). There was no obvious elbow point for clustering the data. This is probably because the districts in Boston all have similar types of venues (cafes, pizza place), especially in southern Boston area. However, based on past experiences and intuition, I went with 4 clusters. Exploring different machine learning algorithms should also be considered in the future. 


Thank you

