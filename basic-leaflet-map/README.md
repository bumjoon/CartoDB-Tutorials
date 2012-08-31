Building a basic Leaflet map using [CartoDB](http://cartodb.com)
== 

This tutorial will walk you through a set of basic steps to build an leaflet map using CartoDB. 

Keep in mind that this tutorial was built for users with paid accounts. Free accounts will not have enough space to use the full United States counties shapefile mentioned below.

To see the final application, go here,

[Live Leaflet Map](http://vizzuality.github.com/CartoDB-Tutorials/basic-leaflet-map/index.html)

## Data sources

#####United States county borders

  - http://www.census.gov/geo/www/cob/co2000.html

  - Download the full shapefile, http://www.census.gov/geo/cob/bdy/co/co00shp/co99_d00_shp.zip

#####Youth in adult facilities

  - http://public.tableausoftware.com/views/CountyandStateJail/Sheet3

  - Export the data, http://vizzuality.github.com/CartoDB-Tutorials/basic-leaflet-map/data/youth_by_county.csv.zip

##Building the map

###Upload your data

1. You can drag both of the files you just downloaded directly onto your CartoDB admin page. From there, they will be uploaded and you should be given tables. 
2. After each of the files is uploaded, be sure that we are all using the same table names, rename the table of your county outlines to "usa_counties" and rename your youth jailed data to "youth_jailed".
3. Go to the Map tab in the usa_counties table and play around with some of the styling options.

###A basic choropleth

1. In your "Map" tab, click the dropdown button beside "Visualization Type".
2. Click the "Numeric Choropleth" radio button
3. For column, choose "area"
4. Select any variation for the rest of the style options.
5. You should now have something similar to,

![basic CartoDB choropleth](http://i.imgur.com/l3L29.png)

###Preparing to merge data

1. Take a look at your youth_jailed table on CartoDB
2. Find the column containing the number of youth jailed in any county (each row is a county).
3. For me, the column is called "__of_juvenile_inmates_reported_in_most_recent_survey" and I don't want to ever type that again, so I'll change the name. Double click the name of your column, in the input field type the new name, "total".
4. Below our column now named "total" click the text that says "string" and change the column type to "number".
5. Finally, we will want to join our two tables based on their geometry. However, our youth_jailed table failed to automatically detect the latitude and longitude columns. Click the "georeference" button at the top right. For a longitude column select "longitude_generated" and for a latitude column select "latitude_generated" then click "Georeference".

###Basic SQL

Play around with some of the following SQL statements

    SELECT sum(total) FROM youth_jailed


    SELECT state,sum(total) FROM youth_jailed GROUP BY state

You may notice in the last query that "MT" has no value in the sum column result. This is because no value existed in the input table. If you want to remove the row, you can do it manually or as follows,

    DELETE FROM youth_jailed WHERE state = 'MT'
    
###Merge two tables

You will very often want to JOIN data from two tables. In this case, we want to know the county outline stored in our usa_counties table and the count of youth in our youth_jailed table.

1. Click at the bottom left "SQL" In the window that appears, run the SQL statement below and take a look at the results.


    SELECT 
      name,count(*) 
    FROM 
      usa_counties, youth_jailed 
    WHERE 
      ST_Intersects(
        youth_jailed.the_geom, 
        usa_counties.the_geom
      ) 
    GROUP BY 
      usa_counties.name

Unlike the previous SQL statements we played with, you'll notice that this statement keeps repeating the table names, like, youth_jailed.something. This is so that columns between the tables are confused, and is only required when the tables we are querying at the same time both have a column named the same thing. In this case, it was needed for "the_geom", which does exist in both. 

###Updating your table

Now, we want to use the data stored in our youth_jailed table with the polygons we uploaded into usa_counties. We could do this on the fly from our HTML map actually using the SQL above, but for this tutorial, we are going to join them once and store the results directly in our usa_counties map.

1. Go back to your usa_counties table. You are going to need someplace to store the value, so let's create a new column called "total_youth". To do so, click the down-arrow beside any column name. In the menu that appears, click "Add new column". From there, add a column called "total_youth" and set its type to "number". Hit Create.
2. Now run an UPDATE SQL statement very similar to the JOIN statement we ran previously,


    UPDATE 
      usa_counties as u
    SET 
      "total_youth" = (
        SELECT 
          count(*)
        FROM
          youth_jailed as y
        WHERE 
          ST_Intersects(
            y.the_geom, 
            u.the_geom
          ) 
        GROUP BY 
          u.name
      )

Finally, you may notice that this leaves a lot of your columns (counties) without a value for total_youth. This is because the jailed_youth table only contains values for counties with < 0. This is fine, and we can actually use it to do some nice styling later.

###Make a new choropleth

Just like you did before, change the style of your usa_counties table. Set it to a choropleth based on this new column that we just created, "total_youth". You should have something like this now, don't mind the gray,

![new choropleth](http://i.imgur.com/e8NMw.png)








Tutorial given in August 2012 by @andrewxhill