---
layout: page
title: Flood Vulnerability in Dar-es-Salaam
---

Created: `24 March 2021`
Revised: `25 March 2021`

#Flood Vulnerability in Dar-es-Salaam

### Question

### Data & data citations & descriptions

### Methods  

  First off, a general building layer is extracted from OSM with our desired attributes. Note that the geometry has to be casted onto the features. This is important because spatial queries will be executed later on.

```sql
CREATE TABLE buildings_all AS
Select building, osm_id, "building:material", way::geometry(polygon,4326) as geom
FROM public.material_polygon Limit 1500000;
```

Now, we extract **homes** layer. To do this we select any building that has any of the attributes listed below. The "yes" attribute may refer to homes, therefore it is included, but should be noted that there is not a 100 percent certainty that buildings with the "yes" attribute are only residential.

```sql
CREATE TABLE homes AS
SELECT "building:material", osm_id, building, st_multi(geom)::geometry(multipolygon,4326) as way
FROM buildings_all
WHERE building IN ( 'yes', 'residential', 'commercial;residential', 'apartments');
```

In this step, a new "vulnerable" attribute is created and filled depending on the material of the building.

```sql
ALTER TABLE homes
ADD COLUMN vulnerable int;

UPDATE homes
SET
	vulnerable = 2
WHERE
	"building:material" = 'wood'
	OR
	"building:material" = 'bamboo'
	OR
	"building:material" = 'earthen'
	OR
	"building:material" = 'waste material';

UPDATE homes
SET
	vulnerable = 0
WHERE
	"building:material" = 'brick'
	OR
	"building:material" = 'cement_block'
	OR
	"building:material" = 'concrete'
	OR
	"building:material" = 'glass'
	OR
	"building:material" = 'plaster'
	OR
	"building:material" = 'metal'
	OR
	"building:material" = 'stone';

  UPDATE homes
    SET
    	vulnerable = 1
    where
    	"building:material" IS NULL;
```
The original geometry of the homes layer was not multipolygon and was not in the appropriate CRS. This is fixed by the following:
```sql
SELECT addgeometrycolumn('your_user','homes','homegeom',32737,'MULTIPOLYGON',2);

UPDATE homes
SET homegeom =  ST_Transform(way, 32737);

ALTER TABLE homes
DROP COLUMN way;
```
With the homes layer with the vulnerability attribute finished, it is time to look at the flooding scenarios and add a risk attribute to each home depending on their location in relation to the flood areas.

```sql
---- FOR FLOOD RISK 2 using 25 an 50 cm scenarios only -----
CREATE TABLE flood_2
AS
SELECT flood.*
FROM flood
WHERE flood.flood_leve = 25 OR flood.flood_leve = 50;

---- FOR FLOOD RISK 1 using 100 cms and above scenarios only ---

CREATE TABLE flood_3
AS
SELECT flood.*
FROM flood
WHERE flood.flood_leve > 99;
```

Then, dissolve the flood areas to speed up analysis and add attribute risk.

```sql
CREATE TABLE flooddissolve_2
AS
SELECT st_union(geom)::geometry(multipolygon,32737) as geom
FROM flood_2;


CREATE TABLE flooddissolve_3
AS
SELECT st_union(geom)::geometry(multipolygon,32737) as geom
FROM flood_3;

```

Now its time to create a two new tables of homes that intersect with each level of flood area to add their respective flood attribute.
```sql

CREATE TABLE homes_risk_2
AS
SELECT homes.*, st_multi(st_intersection(homes.homegeom, flooddissolve_2.geom))::geometry(multipolygon,32737) as geom_h
FROM homes INNER JOIN flooddissolve_2
ON st_intersects(homes.homegeom, flooddissolve_2.geom);

ALTER TABLE homes_risk_2
DROP COLUMN homegeom;

ALTER TABLE homes_risk_2
ADD COLUMN risk int;

UPDATE homes_risk_2
SET risk = 2;

CREATE TABLE homes_risk_3
AS
SELECT homes.*, st_multi(st_intersection(homes.homegeom, flooddissolve_3.geom))::geometry(multipolygon,32737) as geom_h
FROM homes INNER JOIN flooddissolve_3
ON st_intersects(homes.homegeom, flooddissolve_3.geom);

ALTER TABLE homes_risk_3
DROP COLUMN homegeom;

ALTER TABLE homes_risk_3
ADD COLUMN risk int;

UPDATE homes_risk_3
SET risk = 1;

```
Now, we create a new homes table that will have an attribute "risk" and values 2 for high risk of flooding (25-500 cm scenarios), 1 for moderate (100cms and above) and 0 for none.

```sql
CREATE TABLE homes_joined
AS
SELECT homes.*, homes_risk_2.risk
FROM homes LEFT JOIN homes_risk_2
ON homes.osm_id = homes_risk_2.osm_id;

UPDATE homes_joined
SET risk = homes_risk_3.risk
FROM homes_risk_3
WHERE homes_joined.osm_id = homes_risk_3.osm_id;

UPDATE homes_joined
SET risk = 0
WHERE risk IS NULL;
```

### Results
  Interpretation
  Link to leaflet map and static maps
