+++
title = "Configuration"
weight = 4
+++

Configuration
=============

Services can be configured in a text file with [TOML](https://github.com/toml-lang/toml) syntax.

A good starting point is the template generated with the `genconfig` command:

    t_rex genconfig --dbconn postgresql://user:pass@localhost/dbname

Configuration file example:

```toml
[service.mvt]
viewer = true

[[datasource]]
name = "buildings"
dbconn = "postgresql://postgres@127.0.0.1/osm_buildings"

[[datasource]]
name = "natural_earth"
path = "natural_earth.gpkg"

[grid]
predefined = "web_mercator"

[[tileset]]
name = "osm"

[[tileset.layer]]
name = "points"
datasource = "natural_earth"
# Select all attributes of table:
table_name = "ne_10m_populated_places"
geometry_field = "wkb_geometry"
geometry_type = "POINT"

[[tileset.layer]]
name = "buildings"
datasource = "buildings"
geometry_field = "geometry"
geometry_type = "POLYGON"
# Clip polygons with a buffer
buffer_size = 10
simplify = true
  # Queries for different zoom levels:
  [[tileset.layer.query]]
  minzoom = 0
  sql = """
    SELECT name, type, 0 as osm_id, ST_Union(geometry) AS geometry
    FROM osm_buildings_gen0
    WHERE geometry && !bbox!
    GROUP BY name, type
    ORDER BY sum(area) DESC"""
  [[tileset.layer.query]]
  minzoom = 17
  maxzoom = 22
  sql = """
    SELECT name, type, osm_id, geometry
    FROM osm_buildings
    WHERE geometry && !bbox!
    ORDER BY area DESC"""

[cache.file]
base = "/var/cache/mvtcache"

[webserver]
bind = "0.0.0.0"
port = 8080
```

The configuration can be extended with [template](https://tera.netlify.com/docs/templates/) commands.
See [templated.toml](https://github.com/t-rex-tileserver/t-rex/blob/master/examples/templated.toml) as an example.

The configuration can include environment variables in the form of `env.VARNAME`. Example:

```
dbconn = "postgresql://{{env.PGUSER}}:{{env.PGPASSWORD}}@{{env.PGHOST}}/osm_buildings"
```

Environment variables are also loaded from a file named `.env` in the current directory or any of its parents.


### Layer configuration

Custom queries can be configured as PostGIS SQL queries.

The following variables are replaced at runtime:

* `!bbox!`: Bounding box of tile
* `!zoom!`: Zoom level of tile request
* `!scale_denominator!`: Map scale of tile request
* `!pixel_width!`: Width of pixel in grid units

If an `fid_field` is declared, this field is used as the feature ID.

### Custom tile grids

t-rex has two built-in grids, `web_mercator` and `wgs84`. Here's an example showing how to define a custom grid:

```toml
[grid.user]
width = 256
height = 256
extent = { minx = 2420000.0, miny = 1030000.0, maxx = 2900000.0, maxy = 1350000.0 }
srid = 2056
units = "m"
resolutions = [4000.0,3750.0,3500.0,3250.0,3000.0,2750.0,2500.0,2250.0,2000.0,1750.0,1500.0,1250.0,1000.0,750.0,650.0,500.0,250.0,100.0,50.0,20.0,10.0,5.0,2.5,2.0,1.5,1.0,0.5]
origin = "TopLeft"
```
