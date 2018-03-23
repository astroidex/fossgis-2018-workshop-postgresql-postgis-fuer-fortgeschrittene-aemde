# Workshop PostgreSQL/PostGIS für Fortgeschrittene

[FOSSGIS-Konferenz 2018 Bonn](https://www.fossgis-konferenz.de/2018/)

![](img/fossgis-konferenz-2018.png ) ![](img/postgresql_postgis.png)


## Referentin

* Astrid Emde
* WhereGroup GmbH & Co. KG 
* astrid.emde@wheregroup.com
* [@astroidex](https://twitter.com/astroidex)


## Themen 
* Foreign Data Wrapper (FDW)
 * postgres_fdw
 * ogr_fdw
* Funktionen & Trigger
* Window Functions
* PostGIS Advanced
 * was könnte interessant sein?
* CTE
* Was machen die Worker in PostGIS?


## OSGeoLive

![](img/osgeolive.png )

Dieser Workshop verwendet OSGeoLive (https://live.osgeo.org) in der Version 11.0 (August 2017). Dabei handelt es sich um ein auf Lubuntu basierendes System mit über 50 vorinstallierten Softwareprojekten. Außerdem enthält OSGeoLive Beispieldatensätze, die für diesen Workshop herangezogen werden.

OSGeoLive kann über den folgenden Link heruntergeladen werden. Sie können OSGeo installieren, über eine virtuelle Maschine einrichten (empfohlen) oder über einen USB-Stick nutzen.

* Download OSGeoLive Image http://live.osgeo.org/en/download.html
* Deutschsprachige Dokumentation https://live.osgeo.org/de/index.html
* PostGIS Überblick (OSGeoLive Overview) https://live.osgeo.org/de/overview/postgis_overview.html
* PostGIS Erster Einstieg (OSGeoLive Quickstart) https://live.osgeo.org/de/quickstart/postgis_quickstart.html


## aktuelle Software Versionen 

* PostgreSQL 10.3 (Stand 1.3.2018) https://www.postgresql.org/
* PostGIS 2.4.0 (Stand 1.10.2017) http://www.postgis.org/


### OSGeoLive 11.0

* PostgreSQL 9.5.11
* PostGIS 2.3.2

Select version(),postgis_version(),postgis_full_version();

## Daten

* Natural Earth 
 * als Shapes
 * Datenbank: natural_earth2
 * Länder, Bundeslaender, Flüsse usw.
* OpenStreetMap
 * Datenbank: osm_local


## weiterführende Links

* PostGIS in Action (August 2015, 2. Auflage) Regine Obe, Leo Hsu ISBN 9781617291395
* Paul Ramsey Blog Clever Elephant http://blog.cleverelephant.ca/
* Clever Elephant ;) https://www.youtube.com/watch?v=Gw_Q1JClH58
* Postgres OnLine Journal Regine Obe, Leo Hsu http://www.postgresonline.com/
* Modern SQL Blog Markus Winand https://modern-sql.com/slides https://use-the-index-luke.com/
* PGConf.DE 13. April 2018 in Berlin http://2018.pgconf.de/
* PostgreSQL Bücher https://www.postgresql.org/docs/books/
* pgRouting: A Practical Guide (Mai 2017, 2. Auflage) Regine Obe, Leo Hsu ISBN: 9780989421737


## Thema: Foreign Datei Wrapper (FDW)

Mit Hilfe von Foreign Datei Wrappern kann eine Verbindung zu anderen Datenquellen aufgebaut werden. Lesender Zugriff ist mittlerweile möglich je nach Datenquelle.
* folgt dem SQL/MED ("SQL Management of External Data") Standard (ab 2003 Teil des SQL Standards)
* ab 2013 auch schreibender Zugriff möglich

* andere PostgreSQL Datenbank
* ORACLE, MS SQL
* OGR


Links
* https://wiki.postgresql.org/wiki/Foreign_data_wrappers
* https://www.postgresql.org/docs/current/static/postgres-fdw.html


Das Konzept 
1. Extension für FDW für Datenbank aktivieren (postgres_fdw)
2. Foreign Server anlegen
3. User Mapping definieren
4. Foreign Table definieren
5. Daten der Fremdtabelle abfragen

```
pgAdmin: Einblenden der FDW Komponenten über Datei -> Optionen -> Browser -> Anzeige
- Fremdserver
- Fremddaten-Wrapper
- Benutzer-Mapping
- Fremdtabellen
```


### Übung: FDW 

-- 1. Extension für FDW für Datenbank aktivieren

```sql
CREATE EXTENSION postgres_fdw;
```

2. Foreign Server anlegen

```sql
CREATE SERVER pg_fdw_naturalearth 
FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS (host 'localhost', dbname 'natural_earth2', port '5432');
```

3. User Mapping definieren

```sql
CREATE USER MAPPING FOR user SERVER pg_fdw_naturalearth 
OPTIONS (user 'user', password 'user');
```

4. Foreign Table definieren

Referenz auf eine Tabelle

```sql
CREATE FOREIGN TABLE fdw_provinces (
  gid integer,
  adm1_code character varying(10),
  shape_leng numeric,
  shape_area numeric,
  diss_me integer,
  adm1_code_ character varying(10),
  iso_3166_2 character varying(10),
  wikipedia character varying(254),
  sr_sov_a3 character varying(3),
  sr_adm0_a3 character varying(3),
  iso_a2 character varying(2),
  adm0_sr smallint,
  admin0_lab smallint,
  name character varying(100),
  name_alt character varying(200),
  name_local character varying(200),
  type character varying(100),
  type_en character varying(100),
  code_local character varying(50),
  code_hasc character varying(10),
  note character varying(254),
  hasc_maybe character varying(50),
  region character varying(100),
  region_cod character varying(50),
  region_big character varying(200),
  big_code character varying(50),
  provnum_ne integer,
  gadm_level smallint,
  check_me smallint,
  scalerank smallint,
  datarank smallint,
  abbrev character varying(10),
  postal character varying(10),
  area_sqkm double precision,
  sameascity smallint,
  labelrank smallint,
  featurecla character varying(50),
  admin character varying(200),
  name_len integer,
  mapcolor9 smallint,
  mapcolor13 smallint,
  the_geom geometry(MultiPolygon,4326)
)
        SERVER pg_fdw_naturalearth
OPTIONS (schema_name 'public', table_name 'ne_10m_admin_1_states_provinces_shp');
```


IMPORT FOREIGN SCHEMA 

* Erzeugt Foreign Tables für alle Tabellen eines Schemas (ab 9.5). Dieser Befehl erleichtert den Import enorm. 
* Über LIMIT TO oder EXCEPT können Einschränkungen definiert werden.
* https://www.postgresql.org/docs/current/static/sql-importforeignschema.html

```sql
Create schema ne;

IMPORT FOREIGN SCHEMA public
    LIMIT TO (ne_10m_land,ne_10m_lakes )
    FROM SERVER pg_fdw_naturalearth
    INTO ne;
```

```sql
IMPORT FOREIGN SCHEMA remote_schema
    [ { LIMIT TO | EXCEPT } ( table_name [, ...] ) ]
    FROM SERVER server_name
    INTO local_schema
    [ OPTIONS ( option 'value' [, ... ] ) ]
```

```sql
Create view planet_osm_point_province as
SELECT osm_id, access, "addr:housename", "addr:housenumber", "addr:interpolation", 
       admin_level, aerialway, aeroway, amenity, area, barrier, bicycle, 
       brand, bridge, boundary, building, capital, construction, covered, 
       culvert, cutting, denomination, disused, ele, embankment, foot, 
       "generator:source", harbour, highway, historic, horse, intermittent, 
       junction, landuse, layer, leisure, lock, man_made, military, 
       motorcar, op.name, "natural", office, oneway, operator, place, poi, 
       population, power, power_source, public_transport, railway, ref, 
       religion, route, service, shop, sport, surface, toll, tourism, 
       "tower:type", tunnel, water, waterway, wetland, width, wood, 
       z_order, way,
       p.name as province
  FROM public.planet_osm_point op, public.fdw_provinces p
  where st_intersects(way,the_geom);
```

```sql
EXPLAIN ANALYZE Select * from planet_osm_point_province;
```

```sql
"Nested Loop  (cost=100.15..870.33 rows=842 width=1313) (actual time=793.550..940.261 rows=7766 loops=1)"
"  ->  Foreign Scan on fdw_provinces p  (cost=100.00..118.94 rows=298 width=250) (actual time=32.733..799.275 rows=3671 loops=1)"
"  ->  Index Scan using planet_osm_point_index on planet_osm_point op  (cost=0.15..2.51 rows=1 width=1095) (actual time=0.007..0.025 rows=2 loops=3671)"
"        Index Cond: (way && p.the_geom)"
"        Filter: _st_intersects(way, p.the_geom)"
"        Rows Removed by Filter: 0"
"Planning time: 0.478 ms"
"Execution time: 954.761 ms"
```

### Übung: FDW
* Legen Sie einen FDW in der Datenbank osm_local an
* Laden der Erweiterung
* Einbinden der Tabelle ne_10m_admin_0_countries in das Schema ne
* Verschneidung der OSM planet_osm_point mit ne_10m_admin_0_countries


## Thema: OGR Foreign Data Wrapper

*  Über den OGR Foreign Data Wrapper kann auch auf Daten im Dateisystem z.B. Shapes zugegriffen werden.
*  SQLite, SQL Server, ODBC, Shape, MapInfo, CSV, Excel, OpenOffice, OpenStreetMap PBF and XML, OGC WebServices...

* Repository auf GitHub von Paul Ramsey https://github.com/pramsey/pgsql-ogr-fdw (Download und Installation)

1. Es müssen die Erweiterungen postgis und ogr_fdw geladen werden.

```sql
CREATE EXTENSION postgis;
CREATE EXTENSION ogr_fdw;
```

Shapes liegen unter  /home/user/data/natural_earth2/

2. Foreign Server anlegen (verweist auf ein Verzeichnis bei Shapes)

```sql
CREATE SERVER ogr_fdw_ne_shapes 
FOREIGN DATA WRAPPER ogr_fdw 
  OPTIONS (
    datasource '/home/user/data/natural_earth2/',
    format 'ESRI Shapefile' );
```

3. User Mapping entfällt bei Shapes


4. Foreign Table definieren

Referenz auf eine Shapdatei im Dateisystem.

```sql
Create schema shp;

IMPORT FOREIGN SCHEMA "ne_10m_admin_0_countries"
	FROM SERVER ogr_fdw_ne_shapes
    INTO shp;
```

```sql
Select * from shp.ne_10m_admin_0_countries;
```

## Funktionaler Index

* Daten sollten über einen räumlichen Index verfügen.
* BBOX zu jedem Objekt wird abgelegt und bei räumlichen Abfragen verwendet
* Funktionaler Index kann erstellt werden - z.B. mit ST_Transform

```sql
CREATE INDEX gist_baum_geom
ON baum 
USING GIST (ST_Transform(geom,4326));
```

## Typ der Spalte Geometry ändern
```sql
Create table provinces_germany as Select * from ne_10m_admin_1_states_provinces_shp where admin='Germany';

ALTER TABLE public.provinces_germany ALTER COLUMN the_geom type geometry(multiPolygon, 25832) USING st_transform(the_geom,25832);
```

## Thema: Funktionen

In PostgreSQL können Funktionen definiert werden. Diese können in unterschiedlichen Sprachen definiert werden, z.B. SQL, C oder PL/pgSQL (Definition über LANGUAGE plpgsql).

Für PL/pgSQL muss die Erweiterung plpgsql vorliegen (per default aktiviert).

Einfacher Funktionsaufbau:

```sql
CREATE FUNCTION getAreaForBufferedPoint(mygeometry geometry, mybuffer float) 
RETURNS float 
 AS 'SELECT ST_Area(ST_Buffer($1,$2));' 
LANGUAGE 'sql'; 
```

```sql
SELECT getAreaForBufferedPoint(geom,0.5) 
FROM ne_10m_populated_places;
```

### Übung: Funktionen
* Erweiterung der Funktion, so dass eine auf 2 Stellen gerundete Zahl ausgegeben wird.
* Beim INSERT in ne_10m_populated_places soll das Land über eine Funktion gefüllt werden (Spalte adm0name)
* Legen Sie eine neue Spalte in der Tabelle  ne_10m_populated_places mit Namen countryname

```sql
CREATE OR REPLACE FUNCTION getCountryname(mygeometry geometry) 
RETURNS character varying 
 AS 'SELECT c.name from ne_10m_admin_1_states_provinces_shp c where st_distance(c.the_geom,$1)=0;' 
LANGUAGE 'sql'; 
```

```sql
Select getCountryname(the_geom) from 
public.ne_10m_populated_places where gid < 5;
```

```sql
ALTER TABLE ne_10m_populated_places ADD COLUMN countryname varchar;
Update ne_10m_populated_places set countryname = getCountryname(the_geom);
```

## Thema: Trigger

Ein Trigger bezieht sich auf eine Tabelle oder eine Sicht und löst beim einem bestimmten Ereignis z.B. beim Einfügen oder Aktualisieren eines Datensatzes eine Aktion aus. 
Ein Trigger ist mit einer Funktion verknüpft.
* https://www.postgresql.org/docs/9.5/static/plpgsql.html
* https://www.postgresql.org/docs/9.5/static/plpgsql-trigger.html

### Übung

Beim Einfügen eines Punktes in ne_10m_populated_places, soll die Spalte, in der die Länderzugehörigkeit definiert ist (countryname) gesetzt werden.

Vorgehen
1. Definition der Triggerfunktion
2. Definition des Triggers


```sql
Drop FUNCTION setCountryname() CASCADE;
CREATE OR REPLACE FUNCTION setCountryname()
  RETURNS trigger AS
$BODY$
	DECLARE
		r RECORD;
	BEGIN
	
	  SELECT c.name INTO r from ne_10m_admin_1_states_provinces_shp c where st_distance(c.the_geom , NEW.the_geom)= 0; 
	
	    RAISE NOTICE 'Countryname: %' ,  r.name;
	
	    IF r.name IS NOT NULL THEN 
	        NEW.countryname = r.name;
    	    END IF;

  	RETURN NEW; 
	END
$BODY$
LANGUAGE plpgsql VOLATILE
COST 100;
-- VOLATILE - immer anderes Ergebnis
-- STABLE - Ergebnis ist gleich, bei gleicher Eingabe. Caching möglich.
```

```sql
CREATE TRIGGER places_set_on_insert_update
  BEFORE INSERT OR UPDATE
  ON ne_10m_populated_places
  FOR EACH ROW
  EXECUTE PROCEDURE setCountryname();
```

```sql  
INSERT INTO ne_10m_populated_places (the_geom) VALUES (ST_GeomFromText('POINT(7.08 50.72)', 4326));
```

```sql
Select countryname from ne_10m_populated_places order by gid desc limit 1;
```

## Thema: Window Functions 

* Berechnungen über mehrere Spalten, die zu einer Spalte ein Beziehung stehen
* Es wird nicht nur eine Zeile (wie bei GROUP BY) ausgegeben. 
* Ausführung: Window Functions werden nach Group, Aggregat und HAVING ausgeführt
* https://www.postgresql.org/docs/current/static/tutorial-window.html
* http://postgis.net/docs/manual-2.4/PostGIS_Special_Functions_Index.html#PostGIS_Window_Functions


```sql
SELECT 
pop_min, 
pop_max, 
adm0name, 
name,
max(pop_max) OVER (PARTITION BY adm0name) 
FROM ne_10m_populated_places
where adm0name = 'Germany'
order by pop_max desc;
```

Definition über Subselect und GROUP

```sql
  Select gid, pop_min, pop_max, adm0name, name, max from ne_10m_populated_places,
  (
    Select max(pop_max), adm0name as country from ne_10m_populated_places GROUP BY adm0name
  ) as foo
  where adm0name = foo.country
  and adm0name = 'Germany'
  order by max;
```

Definition über  WITH Queries (Common Table Expressions)
* https://www.postgresql.org/docs/9.5/static/queries-with.html
* temporäre Tabellen werden erzeugt

```sql
  WITH 
       a AS (Select adm0name, name, pop_max from ne_10m_populated_places),
       b AS (Select max(pop_max), adm0name as countryname from ne_10m_populated_places GROUP BY adm0name)

   Select a.pop_max,a.name, b.countryname, b.max from a,b where countryname = 'Germany' and a.adm0name = b.countryname
   ORDER BY pop_max desc;
```

Window Functions
* ROW_NUMBER() - EInfügen einer laufenden Nummer
* RANK() 
* DENSE_RANK() 

```sql
SELECT pop_min, pop_max, adm0name, name,
max(pop_max::int) OVER w ,
ROW_NUMBER() OVER w ,
RANK() OVER w,
DENSE_RANK() OVER w
from ne_10m_populated_places
where adm0name = 'Germany'
WINDOW w as (PARTITION BY adm0name order by pop_max::int desc)
ORDER BY rank;
```


## Thema: ST_Subdivide

* Teilt ein Multi-/Polygon in viele einzelne Polygone auf 
* Angabe von max_vertices (default ist 256, nicht kleiner als 8)
* Objekte darf nicht mehr Stützpunkte haben als max_vertices
* http://postgis.net/docs/manual-2.4/ST_Subdivide.html
* ab PostGIS 2.3.0

```sql
Create Table provinces_subdivided as Select name, admin, st_subdivide(the_geom) as  the_geom
from ne_10m_admin_1_states_provinces_shp ;
ALTER TABLE provinces_subdivided ADD COLUMN gid serial PRIMARY KEY;
```

```sql
CREATE INDEX provinces_subdivided_the_geom_gist
  ON public.provinces_subdivided
  USING gist
  (the_geom);
```

* mit Angabe von max_vertices (default ist 256, nicht kleiner als 8)

```sql
Create Table provinces_subdivided as Select name, admin, st_subdivide(the_geom,20) as  the_geom
from ne_10m_admin_1_states_provinces_shp ;
ALTER TABLE provinces_subdivided ADD COLUMN gid serial PRIMARY KEY;
VACUUM ANALYZE;
```

### Übung: ST_Subdivide

* Neue Funktion getCountrynameSubdivided() auf die Tabelle provinces_subdivided
* EXPLAIN Prüfung der Performanz

```sql
CREATE OR REPLACE FUNCTION getCountrynameSubdivided(mygeometry geometry) 
RETURNS character varying 
 AS 'SELECT c.name from provinces_subdivided c where st_intersects(c.the_geom,$1);' 
LANGUAGE 'sql'; 
```

```sql
Select getCountrynameSubdivided(the_geom) from 
public.ne_10m_populated_places where gid < 5;
```

```sql
ALTER TABLE ne_10m_populated_places ADD COLUMN countryname varchar;
Update ne_10m_populated_places set countryname = getCountrynameSubdivided(the_geom);
```

## Thema: ST_GeneratePoints 

* ST_GeneratePoints generiert zufällig verteilte Punkte in einer Fläche (Anzahl entsprechend npoints)
* https://postgis.net/docs/ST_GeneratePoints.html
* ab PostGIS 2.3.0

```sql
Select postgis_version();
Select ST_GeneratePoints(the_geom,100) from ne_10m_admin_1_states_provinces_shp where admin = 'Germany';
```

```sql
CREATE TABLE generierte_punkte_pro_land as
SELECT (ST_Dump(the_geom)).geom::geometry(point,4326) AS the_geom, (ST_Dump(the_geom)).path AS path, 
name,
admin from (
Select ST_GeneratePoints(the_geom,100) as the_geom , admin, name
from ne_10m_admin_1_states_provinces_shp where admin = 'Germany'
) as foo;
ALTER TABLE generierte_punkte_pro_land add column gid serial;
```

```sql
CREATE INDEX generierte_punkte_pro_land_the_geom_gist
  ON public.generierte_punkte_pro_land
  USING gist
  (the_geom);
```

```sql
VACUUM ANALYZE;
```

## Thema:  Was machen die Worker in PostGIS?

* Verfügbar ab PostgreSQL 9.6 für Sequenz Scans, Aggregate, Joins
* Problem: Eine Abfrage lief bis dahin nur auf einem Prozessor
* Untersuchungen von Paul Ramsey - siehe Blog
* Aggregate nutzen mehrere Worker
* teilweise nutzt der Query Planner erst nach Anpassung der COST mehrere Worker

Anpassung der postgresql.conf
* max_worker_processes - maximale Anzahl an Hintergrundprozessen. Default 8.
* max_parallel_workers - maximale Anzahl an Workern für das System. Default 8
* max_parallel_workers_per_gather - 0 deaktiviert parallele Abfragen. Default 2



Blog von Paul Ramsey
* http://blog.cleverelephant.ca/2016/03/parallel-postgis.html
* http://blog.cleverelephant.ca/2017/10/parallel-postgis-2.html
* http://blog.cleverelephant.ca/2017/12/postgis-scaling.html


