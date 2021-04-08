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
SELECT addgeometrycolumn('wilmer','homes','homegeom',32737,'MULTIPOLYGON',2);

UPDATE homes
SET homegeom =  ST_Transform(way, 32737);

ALTER TABLE homes
DROP COLUMN way;
```



### Results
  Interpretation
  Link to leaflet map and static maps
