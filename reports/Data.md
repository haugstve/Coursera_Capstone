## Data

There are the data sources needed for a proof of concept in New York. 
- Neighborhood data 
- Venue data
- Travel time data

Neighborhood data is publicly available from the [web](https://geo.nyu.edu/catalog/nyu_2451_34572). It is possible to download it using the direct link (https://geo.nyu.edu/download/file/nyu-2451-34572-geojson.json). This data gives the borough, name, longitude and latitude of neighborhoods in New&nbsp;York.

Venue data is available using the four square API. The number of request could be a bit high for the sand box account. Testing this as a (proof of concept) POC on a subset of the data is one possible solution.

Travel times using public transport is available using the travel time API. Getting travel times from one neighborhood to another at many different times requires allot of queries. Some limitations during the POC could help on this.