## Introduction 
AirBNB makes it possible to rent apartments in cities. At the moment there are 2000+ apartments in New York available for rent during the first weekend in June. These are located in 122 h. Finding the best deal is far from trivial.

The manual approach for selecting an apartment relies on the filter provided by the webpages. Without prior preferences on the neighborhood the selection process is daunting. Sifting through a few apartments from each neighborhood to get a feel for the local area would take hours. Then when the number of neighborhoods is reduced to a more manageable number the task of picking an  still remains. 

For new  there are no reviews available. Information regarding the neighborhood is provided by the owner. This makes it hard to make informed decisions without already knowing the city. The lack of reviews makes  that are less popular. This is reflected in the rent, which means the best deals are among those apartments. 

Using machine learning to identify key information from different neighborhoods would ease the decision. Choosing the right neighborhood before starting to check out individual apartments can reduce the amount of work with 90%. This leaves more time to identify the best deal.

In remote neighborhoods there are fewer apartments and therefore less information. This makes identifying good deals harder in such locations. However, for many people this might be the best deals. Consider this example: You want to rent in a quite neighborhood while having the option to reach venues of a special category using public transport. It is a complex problem to solve if you have no prior information regarding the city you are visiting.

## Data
There are the data sources needed for a proof of concept in New York. 
- Neighborhood data 
- Venue data
- Travel time data

Neighborhood data is publicly available from the [web](https://geo.nyu.edu/catalog/nyu_2451_34572). It is possible to download it using the direct link (https://geo.nyu.edu/download/file/nyu-2451-34572-geojson.json). This data gives the borough, name, longitude and latitude of neighborhoods in New&nbsp;York.

Venue data is available using the four square API. The number of request could be a bit high for the sand box account. Testing this as a (proof of concept) POC on a subset of the data is one possible solution.

Travel times using public transport is available using the travel time API. Getting travel times from one neighborhood to another at many different times requires allot of queries. Some initial limitations during the POC could help on this.

## Methodology 

### Data selection and summarizing
During exploratory analysis, summarizing the data is essential. During this process some adjustment had to be made in terms during . 

Each data set must be cleaned. The neighborhoods in New York contains the location and the Borough of 306 neighborhoods. Removing neighborhoods with duplicate names and separating out the neighborhoods in Manhattan is the first step. 

Keeping only one of the neighborhoods when multiple neighborhoods have the same name is not optimal. However, there are only 2 pairs of neighborhoods with the same name. This leaves 304 which should be plenty. 

Using the location data for neighborhoods outside of Manhattan, venue data is collected from Four Square. The radius chosen is a five minutes walk time which is about 420 meters. Thinking of distance as time makes it takes makes it easier to compare venues in a local neighborhood to venues you have to travel to reach. The choice of excluding manhattan is the large number of venues in those neighborhoods and the limit on 100 venues for each request. The original intention was to include neighborhoods in Manhattan when calculation venues reachable with public transport. This was not done due to the API limitations.

After loading the four square data the location data is processed a second time. Removing neighborhoods that have no venues makes implementation simpler. There could be users which are interested in neighborhoods with no venues but this is left as future work.

Loading travel times was time consuming. Therefore only a single travel time using public transport was calculated from one neighborhood to another. This means it had to be done 262^2 calls to the API. (The time it takes from A to B gives some indication on the time it takes from B to A but I chose to request information for traveling both ways).

This was calculated for 9&nbsp;AM on a the second friday in 2019. A more precise estimate would use an average from multiple days of the week and different times. It could also be added as a feature to see if a neighborhood is well connected during evenings or weekends.

### Principal component analysis (PCA)
Trying to extract useful information from the data took some time. The concept requires some features that users can sort and filter by. They should be something users care about and the users should find that the neighborhoods match the problem. PCA finds the combination of variables which have the highest variance. It reduces the dimensionality. The hope  is that the resulting components can be used to summarize the data in a way understandable to humans.

To start the process the venue information was summarized by one hot encoding and then  by finding the number of venues in each venue category. This will be called the local data. 

In addition a set of travel data was calculated. For each venue category a weighted sum of the number of venues in that category for all other neighborhoods where calculated. The weight used was 1/(#hours). If the number of hours of travel time is zero you are in the same neighborhood. There is no need to travel and the weighted sum of venues traveled to from the local neighborhood should be zero. 

Tested approaches:
- Fit PCA to local venues first and extract the two principal components (PC). Use the same components on the venues that are weighted with travel
- Fit a separate PCA to both the local venues and the venues weighted with local venues (This is the one presented)
- Merge the data and run a PCA on the combined information

Untested approaches: 
- Sum the local and the venues that er weighted with travel. This has not been tested. This is because most of the value proposition comes from telling a narrative. Therefore, being able to separate the two factors is pivotal.

## Results 
For the local data, results where a bit lacking. Less than 10% of the variance was explained using the two first principal components. Four outliers where identified. These where taken out. Sadly this did not result in a significant increase in explanatory power. Looking at the outliers it is clear that the most important factor is the total number of venues. 

For the travel time weighted venue data there was much more success. 80% of the variance was explained using the two first PC. This means that a computer can summarize most of the information into two numbers. Hopefully a human can understand the summary.

## Discussion 
The dimension reduction for the local data was unsuccessful. Classifying the neighborhoods is an alternative, however it is not clear that the results will be more understandable using this approach. Instead, the total number of local venues was used. This was scaled with the square root to give a more even distribution. Finally the number was normalized using min-max, multiplying by 10 and rounding down to integers. This gives a number from 0 to 10. I named the corresponding scale *Busy*. It might not be the best way of figuring out how busy a neighborhood is, but it gives an indication. 

For the travel time weighted venue data the method was much more successful. The first principal component is almost the same as a sum of travel time weighted venues. The component was transformed using min-max normalization and then multiplied by then. The corresponding scale was named *Connectivity*. It seems like a good name. IF something has high connectivity you can reach allot of venues and that is the correct interpretation. 

For the second principal component the interpretation is not as straight forward. Since the weights are distributed around zero it means some type of venues does not contribute (close to zero), some are positive (0.05 to 0.15) while some are negative (-0.05 to -0.15). Looking at the type of venues which are most positive and most negative the component seems to be related to how urban something is. This interpretation is not perfect and adding more data is likely to change the interpretation. 

### Extensions
The possibilities are numerous. Adding pricing data from AirBNB, crime records, traffic data and review data from FourSquare are only some of the options. As a simple improvement adding in travel time weighted venue data from Manhattan should give a much better picture. 

### Conclusion 
The approach yielded useful insights about neighborhoods. More data and more verification should be included in a production ready solution. As a proof of concept, this project shows that free APIs can be used to make a minimal viable product. 