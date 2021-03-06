## PostgreSQL and OpenStreetMap

While I never used this project on a regular basis, it was created so that I could get a local copy of the OSM data for the UC Santa Cruz. This data can then be used with Mapbox or geojson.io for styling purposes.
Here's what we've got setup and how it works on our Nebula VM system.

### VM
We have an Ubuntu 14.04 instance running in the Nebula service at 128.114.97.209. 

### PostgreSQL
We are running PG 9.3

### Automation
In /home/ucscmap is a shell script called get-ucsc-osm.sh that queries Overpass Turbo, https://overpass-turbo.eu/ to pull down fresh OpenStreetMap data for the campus using a bounding box. 
Data is retrieved to a file with a datestamp in the file name.

### Data Loading
To load this data into PostgreSQL, we use the osm2pgsql command while logged in as the ucscmap user.

```su - ucscmap```
```osm2pgsql -c -ducscmap -Uucscmap -S /usr/share/osm2pgsql/default.style ucsc-osm-05_16_18-22.22.30```

The process will output the log as it imports the data.

```
osm2pgsql SVN version 0.82.0 (64bit id space)
[snipped]
Osm2pgsql took 1s overall
```

### Now What?
With the data loaded, you can use a PG client on your workstation to make a connection to the database to query for various record sets. The resulting dataset can be used with your project as an input to styling data, changing or overriding, etc.

https://wiki.openstreetmap.org/wiki/PostgreSQL

