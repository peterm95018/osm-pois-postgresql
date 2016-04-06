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
Overpass turob has the ability to show you your query in multiple formats. In the code below, I've saved the all-gender-public-restrooms query in OverpassQL format. We can run a cron job that periodically queries the Overpass turbo server for any new or modifided restrooms.

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

Assuming we saved the code above as all-gender-public-restrooms.ql, we could execute the following command and save the output to a local file.

```$ query-overpass all-gender-public-restrooms.ql > final-all-gender-public-restrooms.geojson```

This resulting data file is equivalent to what you'd copy/paste out of the web interface on Overpass turbo. From here we would setup some type frequncy for a cron job. Probably once a day or week would be sufficient to keep up with changes.

```
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "node/1951346733",
      "properties": {
        "type": "node",
        "id": 1951346733,
        "tags": {
          "access": "yes",
          "amenity": "toilets",
          "name": "Communications 124B",
          "unisex": "yes",
          "wheelchair": "yes"
        },
        "relations": [],
        "meta": {}
      },
      "geometry": {
        "type": "Point",
        "coordinates": [
          -122.0612313,
          37.0009284
        ]
      }
    },
    ```

# Workflow
So far, we're able to create a query and return geoJSON results. The data coming out of Overpass turbo does not usually have properties for title and description. Title and description fields are manadatory for a marker to have a tooltip popup in LeafletJS. If we load the geoJSON file into geojson.io, we can edit and add title, description and marker styling. Saving this editd file depends on our intent. That's not always a fast process.

If we just want to read the geoJSON file into a leaflet template, we can. We can't easily import this edited data into OSM. So, we need to tune these steps a bit further so we can have a repeatable and hopefully automated process.

#Workflow v2
- Query Overpass turbo for CSV data
- Load CSV data into prepopulated template w/headers
- copy name column to title column (for popup)
- figure out a default value for description or copy name to description
- save the edited CSV file
- convert it to geoJSON (csv2geojson)
- save converted file to web server and github.
