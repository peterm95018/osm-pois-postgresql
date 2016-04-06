# osm-pois
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

# Automating and Refreshing via Query
Overpass turob has the ability to show you your query in multiple formats. In the code below, I've saved the all-gender-public-restrooms in geoJSON format. I can then execute a cron job that periodically queries the Overpass turbo server for any new or modifided restrooms.

```
[out:json]
[timeout:25]
;
(
  node
    ["amenity"="toilets"]
    ["unisex"="yes"]
    ["access"="yes"]
    (36.98322048717869,-122.07248210906982,37.01033214683092,-122.04428672790527);
  way
    ["amenity"="toilets"]
    ["unisex"="yes"]
    ["access"="yes"]
    (36.98322048717869,-122.07248210906982,37.01033214683092,-122.04428672790527);
  relation
    ["amenity"="toilets"]
    ["unisex"="yes"]
    ["access"="yes"]
    (36.98322048717869,-122.07248210906982,37.01033214683092,-122.04428672790527);
);
out body;
>;
out skel qt;
```

Using the node module query-overpass, we can now run a query and save the output to a local file. Assuming we saved the code above as all-gender-public-restrooms.ql, we could execute the following command.

```query-overpass all-gender-public-restrooms.ql > final-all-gender-public-restrooms.geojson```

This resulting data file is equivalent to what you'd copy/paste out of the web interface on Overpass turbo.

