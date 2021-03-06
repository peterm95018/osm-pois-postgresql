# OpenStreetMap - Points of Interest
OSM maps and geojson datasets of Points of Interest

To extract POI data out of OpenStreetMap, the best tool to use is http://overpass-turbo.eu. 

Start by drawing a bounding box rectangle around the campus. This sets the bounds of the queries. So, if you bound the campus proper, you won't get the Marine Science Campus, 2300 Delaware, 250 Natural Bridges, 1201 Shaffer or 2155 Delaware in your results. 

The most interesting POI's are amenities such as:
cafe
drinking_water
library
bicycle_parking
bicycle_repair_station
parking
atm
theatre
police
post_office
toilets

Using Overpass, we can create queries that can then be exported as geojson. Once the data is in geojson, you can use the http://geojson.io functionality of github to further style and edit your dataset.

<img src="Screen Shot 2016-03-23 at 5.13.19 PM.png">

# Queries
Here's a simple query. I simply used the Overpass wizard and entered cafe. While this can get us some data, we probably want to adjust the query to include cafe or restaraunt.

```
[out:json][timeout:25];
// gather results
(
  // query part for: “cafe”
  node["amenity"="cafe"]({{bbox}});
  way["amenity"="cafe"]({{bbox}});
  relation["amenity"="cafe"]({{bbox}});
);
// print results
out body;
>;
out skel qt;
```
```
[out:json][timeout:25];
// gather results
(
  // query part for: “cafe”
  node["amenity"="cafe"]({{bbox}});
  way["amenity"="cafe"]({{bbox}});
  relation["amenity"="cafe"]({{bbox}});
  // query part for: “restaurant”
  node["amenity"="restaurant"]({{bbox}});
  way["amenity"="restaurant"]({{bbox}});
  relation["amenity"="restaurant"]({{bbox}});
);
// print results
out body;
>;
out skel qt;
```
Once the data is returned, you'll want to export it (not just copy the data link). Why? Exporting can give you the geojson format. Copying the data just gives you json. Different but significant; ymmv.

# Automating Data Pulls
[edited 051618- see link https://github.com/peterm95018/osm-pois-postgresql/blob/master/postgres-osm-server.md for more info on the Nebula VM]

On the host, postgres-osm (128.114.97.209), we have a shell script at /home/ucscmap that can execute a query on Overpass and return the data in a file with a fresh datestamp. The resulting XML file is in OSM format. The OSM formatted file can then be read into PostgreSQL for further work.

In another example, we want to set a bounding box for the main campus and run a query that gets all our buildings and the centerpoint (lat,lng) for each. We'd then want a second query to have a bounding box around the Coastal Science Campus and 2300 Delaware. One issue will be that we'll get some buildings back that are not part of UCSC. 

We can also use some formatting in the query to pull back CSV instead of XML or geojson; depends on needs. Here's a query on all gender restrooms in CSV.
```
/*
This has been generated by the overpass-turbo wizard.
The original search was:
“amenity=toilets and unisex=yes”
*/
[out:csv(::"id",::lat, ::lon, access,name,amenity)][timeout:25];
// gather results
(
  // query part for: “amenity=toilets and unisex=yes”
  node["amenity"="toilets"]["unisex"="yes"]({{bbox}});
  way["amenity"="toilets"]["unisex"="yes"]({{bbox}});
  relation["amenity"="toilets"]["unisex"="yes"]({{bbox}});
);
// print results
out body;
>;
out skel qt;
```
and the results look like this:
```
@id	@lat	@lon	access	name	amenity
1951346834	36.9911636	-122.0647026	yes	Rachel Carson College Student Commons 205	toilets
2046799051	36.9929504	-122.0605625	yes	Music Center 103	toilets
2046990322	36.9947542	-122.0625344	yes	Main Stage	toilets
2050564411	36.9941925	-122.0650032	yes	Porter Academic Building D217B	toilets
```
I found a reference in Stackoverflow on how to get the center point of a building. It requires adding ```out center meta;``` to the end of the query. Here's a short example.

Get all the buildings at Rachel Carson with center points.
```
/*
This has been generated by the overpass-turbo wizard.
The original search was:
“building=*”
*/
[out:json][timeout:25];
// gather results
(
  // query part for: “building=*”
  node["building"]({{bbox}});
  way["building"]({{bbox}});
  relation["building"]({{bbox}});
);
// print results
out body;
out center;
>;
out skel qt;
```
