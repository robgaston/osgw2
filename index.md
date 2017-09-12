title: Open-source Geospatial Web Stack Workshop
class: animation-fade
layout: true

<!-- This slide will serve as the base layout for all your slides -->
.bottom-bar[
  {{title}}
]

---

class: impact

# {{title}}, Day 1
## Sept. 12, GIS Education Center, CCSF Mission Campus

---

# Introductions

## Who am I?

[Rob Gaston](http://www.rgaston.com/)

- Geospatial web application developer, Farallon Geographics (10 years)

- Sometime educator, GIS Education Center & General Assembly

- Cat lover, volunteer and foster with [Cat Town, Oakland](http://www.cattownoakland.org/)

--

## Who are you?

...and what do you hope to get from this workshop?

---

## This workshop will be:

A practical introduction to a powerful stack of free, open-source web tools for geospatial web development.

We're going to focus on a hands-on approach to learning the basics of:

- PostgreSQL/PostGIS (spatially-enabled [RDBMS](https://en.wikipedia.org/wiki/Relational_database_management_system))

- GeoServer (Geospatial data server)

- Leaflet (JavaScript mapping library)

- TurfJS (JavaScript geospatial processing library)

---

## This workshop won't be:

A thorough academic explanation of GIS or related concepts.  We're not going to spend a lot of time talking about the basics of:

- GIS

- RDBMS/SQL

- JavaScript

- HTML/CSS

...but we're going to try to keep things simple and easy, so don't worry if you're not an expert - some basic comfort and confidence will suffice.

---

## As we're working...

- these slides are viewable at [http://rgaston.com/osgw2/](http://rgaston.com/osgw2/)

- you are encouraged to follow along on your laptop

- there are helpful links throughout that you may want to reference during the exercise

- but, please, try not to jump ahead

- please refrain from starting on the hands-on exercises until I come to those slides or working on the exercises while we're talking

- if you finish a particular exercise and others are still working, find a student who is still working to help (it's really the *best* way to learn and build confidence)

- be nice to each other!

---

class: impact

# Part 1: PostgreSQL & PostGIS basics

---

## What is PostgreSQL?

[PostgreSQL](https://www.postgresql.org/) is a free and open-source [relational database management system (RDBMS)](https://en.wikipedia.org/wiki/Relational_database_management_system), first released in 1996.  It is powerful, secure and fully featured while also being easy to install and administer.

--

Other popular RDBMS's include:

- MySQL (also open-source)

- Oracle

- Microsoft SQL Server

--

RDBMS's provide for the management of relational data tables through the use of foreign key relationships.

Advantages over flat-files beyond a relational model include scalability and security.

The language for querying RDBMS's is called [Structured Query Language (SQL)](https://en.wikipedia.org/wiki/SQL).

---

## What is PostGIS?

[PostGIS](http://postgis.net/) is the geospatial extension for PostgreSQL, it provides:

--

- ability to store geospatial data in tables using the "geometry" data type
    
    - high degree of flexibility here as compared to shapefiles
    
    - a single geometry column can store geometries of many types ([multi-]points, lines, or polygons).
    
    - geometries stored in the same column should use the same spatial reference system

--

- lots of [functions](https://postgis.net/docs/reference.html) for working with geospatial data, including:
    
    - transformation (such as between different spatial reference systems)
    
    - output formatting
    
    - spatial querying

---

## Installing PostgreSQL

PostgreSQL installation can be tricky, as there are many paths.  For this workshop we will use:

- For MacOS: [Postgres.app](http://postgresapp.com/)

- For Windows: [EnterpriseDB Installer w/ Stackbuilder](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads#windows)

We'll be using these tools because:

1. they are easy to install

2. they include the PostGIS extension, so we won't have to install it separately.

Don't rush out and install these right this second - we're going to do that in just a minute.

---

## Working with PostgreSQL

There are lots of tools for working with PostgreSQL and PostGIS.  The most common are:

- the GUI client, [PgAdmin4](https://www.pgadmin.org/download/)

    - shows a nice tree view of database entities
    
    - simple tools for querying data using SQL.

    - it is what we will be using during this workshop.
    
- the command line client, `psql` (it comes with PostgreSQL)
    
    - allows execution of SQL against PostgreSQL databases from a command line
    
- the GIS client, [QGIS](http://www.qgis.org/)

    - does a lot more than just PostGIS, but they go together well

---

## Creating a spatially-enabled database (part 1)

To create a spatially-enabled database in PostgreSQL using PgAdmin4:

- if you haven't, create a connection to your local PostgreSQL instance

    - give your connection a "Name" on the "General" tab (something like "localhost")

    - specify the "Host name/adress" as "localhost" on the "Connection" tab

- right-click on the connection name and select "Create" > "Database"

    - give your database a name (for example, "osgw") and hit "Save"
    
---

## Creating a spatially-enabled database (part 2)

To enable PostGIS on your database:

- right-click on your database's name in the tree view on the left and select "Query Tool"

- paste the following SQL into the resulting SQL query window, and press the lightning bolt button or f5 to run it:

```SQL
-- Enable PostGIS (includes raster)
CREATE EXTENSION postgis;
-- Enable Topology
CREATE EXTENSION postgis_topology;
```

- you can verify that PostGIS is enabled on your database by running the following SQL:

```SQL
SELECT postgis_version();
```

---

class: impact

## Demo: Creating a spatially-enabled database with PgAdmin4

---

class: impact

## Hands-on Exercise

---

## Ex. 1: PostgreSQL & PostGIS setup

On your laptop, please do the following:

1. install [PostgreSQL 9.6 with PostGIS 2](#14) (and command line tools) 

1. install [PgAdmin4](https://www.pgadmin.org/download/) (on Windows, you should be able to do this within the Stackbuilder)

1. using PgAdmin4:

    1. create a new database
    
    1. add the PostGIS extension to your database
    
    1. confirm PostGIS is enabled on your database

1. **extra credit**: if you finish early (or already have these installed), find someone who is still working and help them along

---

class: impact

## 10 minute break

---

class: impact

# Part 2: Loading and querying geospatial data with PostGIS

---

## Loading geospatial data into PostgreSQL

- Geospatial source data are often passed around using the shapefile format.

- PostGIS provides a command line tool for creating a table from a source shapefile called `shp2pgsql`

    - `shp2pgsql` generates the SQL necessary to load your shapefile into a PostgreSQL table
    
    - usage ([reference](http://suite.opengeo.org/docs/latest/dataadmin/pgGettingStarted/shp2pgsql.html)):
    
        `shp2pgsql -I -s <SRID> <PATH/TO/SHAPEFILE> <SCHEMA>.<DBTABLE> | psql -U postgres -d <DBNAME>`
        
    - for example: given a shapefile, `~/Downloads/states/states.shp`, I could load it into a new a table called `states` in my database `osgw` like so:
    
        `shp2pgsql -I -s 4326 ~/Downloads/states/states.shp public.states | psql -U postgres -d osgw`

---

## Querying data with PgAdmin4

- With PgAdmin4, you can view the rows in a table either by:

    1. right clicking on your table in the tree view and selecting "View Data" > "View All Rows"
    
    1. creating a new query with the following SQL (using the name of your table):
        
        `select * from public.states;`
--
        
- Unfortunately, there is no "map" view currently available in PgAdmin4

- Geometry values shown in PgAdmin4 are not human-readable; however, they can be recast using [PostGIS functions](https://postgis.net/docs/reference.html)...


---

## Using PostGIS geospatial functions

- [PostGIS functions](https://postgis.net/docs/reference.html) are a powerful set of tools for working with geospatial data

- They can be called directly in your SQL queries

--

- For example, you can recast geometries in a select to a more human-readable [WKT](https://en.wikipedia.org/wiki/Well-known_text) format like so:
    
    `select *, st_astext(geom) as wkt from public.states;`
    
--
    
- as mentioned, PostGIS functions allow for lots of geospatial operations, such as

    - transformation (for example, buffering, or transforming between spatial reference systems)
    
    - output formatting (as done in the above example with `st_astext`)
    
    - spatial querying (for example, joining data from multiple tables with a spatial intersection)

---

class: impact

## Demo: Adding a shapefile to PostgreSQL and querying with PostGIS

---

class: impact

## Hands-on Exercise

---

## Ex. 2: Loading and querying Geospatial data with PostGIS

1. download the countries and places shapefiles from [Natural Earth](http://www.naturalearthdata.com/downloads/10m-cultural-vectors)

1. using `shp2pgsql`, load these two shapefiles into PostgreSQL

1. using PgAdmin4:

    1. `select * from` your table to view the data that you just loaded
    
    1. write a SQL query to join places to countries using a spatial intersection to get a count of place points in each country, **hint:** you'll need to use a `join` and a `group by` in your query.  I'll let you figure out which PostGIS function tells you if a polygon *contains* a point.
    
1. **extra credit**: how else might we use PostGIS spatial processing functions to ask questions about these data? Come up with a question to ask about these data (or others) through spatial processing, find the right PostGIS functions (I can help you) and write a SQL query to get your answer

---

class: impact

## 10 minute break

---

class: impact

# Part 3: Visualizing PostGIS data on desktop

---

## PostGIS and desktop

- like most spatially-enabled RDBMS, PostgreSQL is supported by most desktop platforms (in one way or another)

- since we are focused on open-source tools, we're going to use [QGIS](http://www.qgis.org/en/site/forusers/download.html)

- [QGIS](http://www.qgis.org/en/site/forusers/download.html) is a powerful free and open-source desktop GIS that makes visualizing PostGIS data simple

---

## PostgreSQL Views

- PostgreSQL (like any RDBMS) allows us to create [views](https://en.wikipedia.org/wiki/View_%28SQL%29)

- database views are entities that can store queries and return their results

- you can select from views just as you would from a database table

- QGIS allows you to create layers from PostGIS views, which is a nice way to visualize the result of spatial queries

---

class: impact

## Demo: Creating a PostgreSQL view, visualizing in QGIS

---

class: impact

## Hands-on Exercise

---

## Ex. 4: Visualizing PostGIS layers in QGIS

On your laptop, do the following:

1. install [QGIS](http://www.qgis.org/en/site/forusers/download.html)

1. create a new view in PostgreSQL using the join query that we wrote in [Ex. 2](#31)

1. start QGIS, and:

    1. create a new connection pointing to your local database
    
    1. add a "countries" layer from the view that you created above

1. **extra credit 1**: restyle your "countries" layer in QGIS 

1. **extra credit 2**: add a layer using another PostGIS table or view; restyle that layer

---

## Homework:

In the time between now and Day 2 of the workshop (July 31), please do the following:

1. find a shapefile containing data that you would like to load into PostgreSQL

1. load your shapefile into a PostgreSQL table

1. visualize and style your PostGIS table in QGIS

---
class: impact

# Thanks - see you on Sept. 19!

...next time: connecting GeoServer to PostgreSQL and viewing data on the web...

