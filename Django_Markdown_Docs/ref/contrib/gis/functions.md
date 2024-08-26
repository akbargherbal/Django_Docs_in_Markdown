# Geographic Database Functions

The functions documented on this page allow users to access geographic database
functions to be used in annotations, aggregations, or filters in Django.

Example:

```pycon
>>> from django.contrib.gis.db.models.functions import Length
>>> Track.objects.annotate(length=Length("line")).filter(length__gt=100)
```

Not all backends support all functions, so refer to the documentation of each
function to see if your database backend supports the function you want to use.
If you call a geographic function on a backend that doesn’t support it, you’ll
get a `NotImplementedError` exception.

Function’s summary:

| Measurement                                                                    | Relationships                                                                                                                                                                                                                  | Operations                                                               | Editors                                                                                                                                                                                                      | Input format                                                                                                                  | Output format                                                                                                                                                                          | Miscellaneous                                                            |
|--------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| [`Area`](#django.contrib.gis.db.models.functions.Area)                         | [`Azimuth`](#django.contrib.gis.db.models.functions.Azimuth)                                                                                                                                                                   | [`Difference`](#django.contrib.gis.db.models.functions.Difference)       | [`ForcePolygonCW`](#django.contrib.gis.db.models.functions.ForcePolygonCW)                                                                                                                                   |                                                                                                                               | [`AsGeoJSON`](#django.contrib.gis.db.models.functions.AsGeoJSON)                                                                                                                       | [`IsEmpty`](#django.contrib.gis.db.models.functions.IsEmpty)             |
| [`Distance`](#django.contrib.gis.db.models.functions.Distance)                 | [`BoundingCircle`](#django.contrib.gis.db.models.functions.BoundingCircle)                                                                                                                                                     | [`Intersection`](#django.contrib.gis.db.models.functions.Intersection)   | [`MakeValid`](#django.contrib.gis.db.models.functions.MakeValid)                                                                                                                                             |                                                                                                                               | [`AsGML`](#django.contrib.gis.db.models.functions.AsGML)                                                                                                                               | [`IsValid`](#django.contrib.gis.db.models.functions.IsValid)             |
| [`GeometryDistance`](#django.contrib.gis.db.models.functions.GeometryDistance) | [`Centroid`](#django.contrib.gis.db.models.functions.Centroid)                                                                                                                                                                 | [`SymDifference`](#django.contrib.gis.db.models.functions.SymDifference) | [`Reverse`](#django.contrib.gis.db.models.functions.Reverse)                                                                                                                                                 |                                                                                                                               | [`AsKML`](#django.contrib.gis.db.models.functions.AsKML)                                                                                                                               | [`MemSize`](#django.contrib.gis.db.models.functions.MemSize)             |
| [`Length`](#django.contrib.gis.db.models.functions.Length)                     | [`ClosestPoint`](#django.contrib.gis.db.models.functions.ClosestPoint)                                                                                                                                                         | [`Union`](#django.contrib.gis.db.models.functions.Union)                 | [`Scale`](#django.contrib.gis.db.models.functions.Scale)                                                                                                                                                     |                                                                                                                               | [`AsSVG`](#django.contrib.gis.db.models.functions.AsSVG)                                                                                                                               | [`NumGeometries`](#django.contrib.gis.db.models.functions.NumGeometries) |
| [`Perimeter`](#django.contrib.gis.db.models.functions.Perimeter)               | [`Envelope`](#django.contrib.gis.db.models.functions.Envelope)<br/>[`LineLocatePoint`](#django.contrib.gis.db.models.functions.LineLocatePoint)<br/>[`PointOnSurface`](#django.contrib.gis.db.models.functions.PointOnSurface) |                                                                          | [`SnapToGrid`](#django.contrib.gis.db.models.functions.SnapToGrid)<br/>[`Transform`](#django.contrib.gis.db.models.functions.Transform)<br/>[`Translate`](#django.contrib.gis.db.models.functions.Translate) | [`FromWKB`](#django.contrib.gis.db.models.functions.FromWKB)<br/>[`FromWKT`](#django.contrib.gis.db.models.functions.FromWKT) | [`AsWKB`](#django.contrib.gis.db.models.functions.AsWKB)<br/>[`AsWKT`](#django.contrib.gis.db.models.functions.AsWKT)<br/>[`GeoHash`](#django.contrib.gis.db.models.functions.GeoHash) | [`NumPoints`](#django.contrib.gis.db.models.functions.NumPoints)         |

## `Area`

### *class* Area(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-polygon-property-functions.html#function_st-area),
Oracle, [PostGIS](https://postgis.net/docs/ST_Area.html), SpatiaLite

Accepts a single geographic field or expression and returns the area of the
field as an [`Area`](measure.md#django.contrib.gis.measure.Area) measure.

MySQL and SpatiaLite without LWGEOM/RTTOPO don’t support area calculations on
geographic SRSes.

## `AsGeoJSON`

### *class* AsGeoJSON(expression, bbox=False, crs=False, precision=8, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-geojson-functions.html#function_st-asgeojson),
Oracle, [PostGIS](https://postgis.net/docs/ST_AsGeoJSON.html), SpatiaLite

Accepts a single geographic field or expression and returns a [GeoJSON](https://geojson.org/) representation of the geometry. Note that the result
is not a complete GeoJSON structure but only the `geometry` key content of a
GeoJSON structure. See also [GeoJSON Serializer](serializers.md).

Example:

```pycon
>>> City.objects.annotate(json=AsGeoJSON("point")).get(name="Chicago").json
{"type":"Point","coordinates":[-87.65018,41.85039]}
```

| Keyword Argument   | Description                                                                                                                                                           |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `bbox`             | Set this to `True` if you want the bounding box<br/>to be included in the returned GeoJSON. Ignored on<br/>Oracle.                                                    |
| `crs`              | Set this to `True` if you want the coordinate<br/>reference system to be included in the returned<br/>GeoJSON. Ignored on MySQL and Oracle.                           |
| `precision`        | It may be used to specify the number of significant<br/>digits for the coordinates in the GeoJSON<br/>representation – the default value is 8. Ignored on<br/>Oracle. |

## `AsGML`

### *class* AsGML(expression, version=2, precision=8, \*\*extra)

*Availability*: Oracle, [PostGIS](https://postgis.net/docs/ST_AsGML.html),
SpatiaLite

Accepts a single geographic field or expression and returns a [Geographic Markup
Language (GML)](https://en.wikipedia.org/wiki/Geography_Markup_Language) representation of the geometry.

Example:

```pycon
>>> qs = Zipcode.objects.annotate(gml=AsGML("poly"))
>>> print(qs[0].gml)
<gml:Polygon srsName="EPSG:4326"><gml:OuterBoundaryIs>-147.78711,70.245363 ...
-147.78711,70.245363</gml:OuterBoundaryIs></gml:Polygon>
```

| Keyword Argument   | Description                                                                                                                                   |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| `precision`        | Specifies the number of significant digits for the<br/>coordinates in the GML representation – the default<br/>value is 8. Ignored on Oracle. |
| `version`          | Specifies the GML version to use: 2 (default) or 3.                                                                                           |

## `AsKML`

### *class* AsKML(expression, precision=8, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_AsKML.html), SpatiaLite

Accepts a single geographic field or expression and returns a [Keyhole Markup
Language (KML)](https://developers.google.com/kml/documentation/) representation of the geometry.

Example:

```pycon
>>> qs = Zipcode.objects.annotate(kml=AsKML("poly"))
>>> print(qs[0].kml)
<Polygon><outerBoundaryIs><LinearRing><coordinates>-103.04135,36.217596,0 ...
-103.04135,36.217596,0</coordinates></LinearRing></outerBoundaryIs></Polygon>
```

| Keyword Argument   | Description                                                                                                                                          |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `precision`        | This keyword may be used to specify the number of<br/>significant digits for the coordinates in the KML<br/>representation – the default value is 8. |

## `AsSVG`

### *class* AsSVG(expression, relative=False, precision=8, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_AsSVG.html), SpatiaLite

Accepts a single geographic field or expression and returns a [Scalable Vector
Graphics (SVG)](https://www.w3.org/Graphics/SVG/) representation of the geometry.

| Keyword Argument   | Description                                                                                                                                                |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `relative`         | If set to `True`, the path data will be implemented<br/>in terms of relative moves. Defaults to `False`,<br/>meaning that absolute moves are used instead. |
| `precision`        | This keyword may be used to specify the number of<br/>significant digits for the coordinates in the SVG<br/>representation – the default value is 8.       |

## `AsWKB`

### *class* AsWKB(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-format-conversion-functions.html#function_st-asbinary),
Oracle, [PostGIS](https://postgis.net/docs/ST_AsBinary.html), SpatiaLite

Accepts a single geographic field or expression and returns a [Well-known
binary (WKB)](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry#Well-known_binary) representation of the geometry.

Example:

```pycon
>>> bytes(City.objects.annotate(wkb=AsWKB("point")).get(name="Chelyabinsk").wkb)
b'\x01\x01\x00\x00\x00]3\xf9f\x9b\x91K@\x00X\x1d9\xd2\xb9N@'
```

## `AsWKT`

### *class* AsWKT(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-format-conversion-functions.html#function_st-astext),
Oracle, [PostGIS](https://postgis.net/docs/ST_AsText.html), SpatiaLite

Accepts a single geographic field or expression and returns a [Well-known text
(WKT)](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) representation of the geometry.

Example:

```pycon
>>> City.objects.annotate(wkt=AsWKT("point")).get(name="Chelyabinsk").wkt
'POINT (55.137555 61.451728)'
```

## `Azimuth`

### *class* Azimuth(point_a, point_b, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Azimuth.html),
SpatiaLite (LWGEOM/RTTOPO)

Returns the azimuth in radians of the segment defined by the given point
geometries, or `None` if the two points are coincident. The azimuth is angle
referenced from north and is positive clockwise: north = `0`; east = `π/2`;
south = `π`; west = `3π/2`.

## `BoundingCircle`

### *class* BoundingCircle(expression, num_seg=48, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_MinimumBoundingCircle.html),
[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/spatl/SDO_GEOM-reference.html#GUID-82A61626-BB64-4793-B53D-A0DBEC91831A),
SpatiaLite 5.1+

Accepts a single geographic field or expression and returns the smallest circle
polygon that can fully contain the geometry.

The `num_seg` parameter is used only on PostGIS.

#### Versionchanged
SpatiaLite 5.1+ support was added.

## `Centroid`

### *class* Centroid(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-polygon-property-functions.html#function_st-centroid),
[PostGIS](https://postgis.net/docs/ST_Centroid.html), Oracle, SpatiaLite

Accepts a single geographic field or expression and returns the `centroid`
value of the geometry.

## `ClosestPoint`

### *class* ClosestPoint(expr1, expr2, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_ClosestPoint.html),
SpatiaLite

Accepts two geographic fields or expressions and returns the 2-dimensional
point on geometry A that is closest to geometry B.

## `Difference`

### *class* Difference(expr1, expr2, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-operator-functions.html#function_st-difference),
[PostGIS](https://postgis.net/docs/ST_Difference.html), Oracle, SpatiaLite

Accepts two geographic fields or expressions and returns the geometric
difference, that is the part of geometry A that does not intersect with
geometry B.

## `Distance`

### *class* Distance(expr1, expr2, spheroid=None, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-relation-functions-object-shapes.html#function_st-distance),
[PostGIS](https://postgis.net/docs/ST_Distance.html), Oracle, SpatiaLite

Accepts two geographic fields or expressions and returns the distance between
them, as a [`Distance`](measure.md#django.contrib.gis.measure.Distance) object. On MySQL, a raw
float value is returned when the coordinates are geodetic.

On backends that support distance calculation on geodetic coordinates, the
proper backend function is automatically chosen depending on the SRID value of
the geometries (e.g. [ST_DistanceSphere](https://postgis.net/docs/ST_DistanceSphere.html) on PostGIS).

When distances are calculated with geodetic (angular) coordinates, as is the
case with the default WGS84 (4326) SRID, you can set the `spheroid` keyword
argument to decide if the calculation should be based on a simple sphere (less
accurate, less resource-intensive) or on a spheroid (more accurate, more
resource-intensive).

In the following example, the distance from the city of Hobart to every other
[`PointField`](model-api.md#django.contrib.gis.db.models.PointField) in the `AustraliaCity`
queryset is calculated:

```pycon
>>> from django.contrib.gis.db.models.functions import Distance
>>> pnt = AustraliaCity.objects.get(name="Hobart").point
>>> for city in AustraliaCity.objects.annotate(distance=Distance("point", pnt)):
...     print(city.name, city.distance)
...
Wollongong 990071.220408 m
Shellharbour 972804.613941 m
Thirroul 1002334.36351 m
...
```

#### NOTE
Because the `distance` attribute is a
[`Distance`](measure.md#django.contrib.gis.measure.Distance) object, you can easily express
the value in the units of your choice. For example, `city.distance.mi` is
the distance value in miles and `city.distance.km` is the distance value
in kilometers. See [Measurement Objects](measure.md) for usage details and the list of
[Supported units](measure.md#supported-units).

## `Envelope`

### *class* Envelope(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-general-property-functions.html#function_st-envelope),
[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/spatl/spatial-operators-reference.html#GUID-ACED800F-3435-44AA-9606-D40934A23ED0),
[PostGIS](https://postgis.net/docs/ST_Envelope.html), SpatiaLite

Accepts a single geographic field or expression and returns the geometry
representing the bounding box of the geometry.

## `ForcePolygonCW`

### *class* ForcePolygonCW(expression, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_ForcePolygonCW.html),
SpatiaLite

Accepts a single geographic field or expression and returns a modified version
of the polygon/multipolygon in which all exterior rings are oriented clockwise
and all interior rings are oriented counterclockwise. Non-polygonal geometries
are returned unchanged.

## `FromWKB`

### *class* FromWKB(expression, srid=0, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-wkb-functions.html#function_st-geomfromwkb),
Oracle, [PostGIS](https://postgis.net/docs/ST_GeomFromWKB.html), SpatiaLite

Creates geometry from [Well-known binary (WKB)](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry#Well-known_binary) representation. The optional
`srid` argument allows to specify the SRID of the resulting geometry.
`srid` is ignored on Oracle.

#### Versionchanged
The `srid` argument was added.

## `FromWKT`

### *class* FromWKT(expression, srid=0, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-wkt-functions.html#function_st-geomfromtext),
Oracle, [PostGIS](https://postgis.net/docs/ST_GeomFromText.html), SpatiaLite

Creates geometry from [Well-known text (WKT)](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) representation. The optional
`srid` argument allows to specify the SRID of the resulting geometry.
`srid` is ignored on Oracle.

#### Versionchanged
The `srid` argument was added.

## `GeoHash`

### *class* GeoHash(expression, precision=None, \*\*extra)

*Availability*: [MySQL](https://dev.mysql.com/doc/refman/en/spatial-geohash-functions.html#function_st-geohash),
[PostGIS](https://postgis.net/docs/ST_GeoHash.html), SpatiaLite
(LWGEOM/RTTOPO)

Accepts a single geographic field or expression and returns a [GeoHash](https://en.wikipedia.org/wiki/Geohash)
representation of the geometry.

The `precision` keyword argument controls the number of characters in the
result.

## `GeometryDistance`

### *class* GeometryDistance(expr1, expr2, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/geometry_distance_knn.html)

Accepts two geographic fields or expressions and returns the distance between
them. When used in an [`order_by()`](../../models/querysets.md#django.db.models.query.QuerySet.order_by) clause,
it provides index-assisted nearest-neighbor result sets.

## `Intersection`

### *class* Intersection(expr1, expr2, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-operator-functions.html#function_st-intersection),
[PostGIS](https://postgis.net/docs/ST_Intersection.html), Oracle, SpatiaLite

Accepts two geographic fields or expressions and returns the geometric
intersection between them.

## `IsEmpty`

### *class* IsEmpty(expr)

*Availability*: [PostGIS](https://postgis.net/docs/ST_IsEmpty.html)

Accepts a geographic field or expression and tests if the value is an empty
geometry. Returns `True` if its value is empty and `False` otherwise.

## `IsValid`

### *class* IsValid(expr)

*Availability*: [MySQL](https://dev.mysql.com/doc/refman/en/spatial-convenience-functions.html#function_st-isvalid),
[PostGIS](https://postgis.net/docs/ST_IsValid.html), Oracle, SpatiaLite

Accepts a geographic field or expression and tests if the value is well formed.
Returns `True` if its value is a valid geometry and `False` otherwise.

## `Length`

### *class* Length(expression, spheroid=True, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-linestring-property-functions.html#function_st-length),
Oracle, [PostGIS](https://postgis.net/docs/ST_Length.html), SpatiaLite

Accepts a single geographic linestring or multilinestring field or expression
and returns its length as a [`Distance`](measure.md#django.contrib.gis.measure.Distance)
measure.

On PostGIS and SpatiaLite, when the coordinates are geodetic (angular), you can
specify if the calculation should be based on a simple sphere (less
accurate, less resource-intensive) or on a spheroid (more accurate, more
resource-intensive) with the `spheroid` keyword argument.

MySQL doesn’t support length calculations on geographic SRSes.

## `LineLocatePoint`

### *class* LineLocatePoint(linestring, point, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_LineLocatePoint.html),
SpatiaLite

Returns a float between 0 and 1 representing the location of the closest point on
`linestring` to the given `point`, as a fraction of the 2D line length.

## `MakeValid`

### *class* MakeValid(expr)

*Availability*: [PostGIS](https://postgis.net/docs/ST_MakeValid.html),
SpatiaLite (LWGEOM/RTTOPO)

Accepts a geographic field or expression and attempts to convert the value into
a valid geometry without losing any of the input vertices. Geometries that are
already valid are returned without changes. Simple polygons might become a
multipolygon and the result might be of lower dimension than the input.

## `MemSize`

### *class* MemSize(expression, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_MemSize.html)

Accepts a single geographic field or expression and returns the memory size
(number of bytes) that the geometry field takes.

## `NumGeometries`

### *class* NumGeometries(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-geometrycollection-property-functions.html#function_st-numgeometries),
[PostGIS](https://postgis.net/docs/ST_NumGeometries.html), Oracle,
SpatiaLite

Accepts a single geographic field or expression and returns the number of
geometries if the geometry field is a collection (e.g., a `GEOMETRYCOLLECTION`
or `MULTI*` field). Returns 1 for single geometries.

On MySQL, returns `None` for single geometries.

## `NumPoints`

### *class* NumPoints(expression, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/gis-linestring-property-functions.html#function_st-numpoints),
[PostGIS](https://postgis.net/docs/ST_NPoints.html), Oracle, SpatiaLite

Accepts a single geographic field or expression and returns the number of points
in a geometry.

On MySQL, returns `None` for any non-`LINESTRING` geometry.

## `Perimeter`

### *class* Perimeter(expression, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Perimeter.html),
Oracle, SpatiaLite

Accepts a single geographic field or expression and returns the perimeter of the
geometry field as a [`Distance`](measure.md#django.contrib.gis.measure.Distance) object.

## `PointOnSurface`

### *class* PointOnSurface(expression, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_PointOnSurface.html),
MariaDB, Oracle, SpatiaLite

Accepts a single geographic field or expression and returns a `Point` geometry
guaranteed to lie on the surface of the field; otherwise returns `None`.

## `Reverse`

### *class* Reverse(expression, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Reverse.html), Oracle,
SpatiaLite

Accepts a single geographic field or expression and returns a geometry with
reversed coordinates.

## `Scale`

### *class* Scale(expression, x, y, z=0.0, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Scale.html), SpatiaLite

Accepts a single geographic field or expression and returns a geometry with
scaled coordinates by multiplying them with the `x`, `y`, and optionally
`z` parameters.

## `SnapToGrid`

### *class* SnapToGrid(expression, \*args, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_SnapToGrid.html),
SpatiaLite

Accepts a single geographic field or expression and returns a geometry with all
points snapped to the given grid.  How the geometry is snapped to the grid
depends on how many numeric (either float, integer, or long) arguments are
given.

|   Number of Arguments | Description                                      |
|-----------------------|--------------------------------------------------|
|                     1 | A single size to snap both the X and Y grids to. |
|                     2 | X and Y sizes to snap the grid to.               |
|                     4 | X, Y sizes and the corresponding X, Y origins.   |

## `SymDifference`

### *class* SymDifference(expr1, expr2, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-operator-functions.html#function_st-symdifference),
[PostGIS](https://postgis.net/docs/ST_SymDifference.html), Oracle,
SpatiaLite

Accepts two geographic fields or expressions and returns the geometric
symmetric difference (union without the intersection) between the given
parameters.

## `Transform`

### *class* Transform(expression, srid, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Transform.html),
Oracle, SpatiaLite

Accepts a geographic field or expression and a SRID integer code, and returns
the transformed geometry to the spatial reference system specified by the
`srid` parameter.

#### NOTE
What spatial reference system an integer SRID corresponds to may depend on
the spatial database used.  In other words, the SRID numbers used for Oracle
are not necessarily the same as those used by PostGIS.

## `Translate`

### *class* Translate(expression, x, y, z=0.0, \*\*extra)

*Availability*: [PostGIS](https://postgis.net/docs/ST_Translate.html),
SpatiaLite

Accepts a single geographic field or expression and returns a geometry with
its coordinates offset by the `x`, `y`, and optionally `z` numeric
parameters.

## `Union`

### *class* Union(expr1, expr2, \*\*extra)

*Availability*: MariaDB, [MySQL](https://dev.mysql.com/doc/refman/en/spatial-operator-functions.html#function_st-union),
[PostGIS](https://postgis.net/docs/ST_Union.html), Oracle, SpatiaLite

Accepts two geographic fields or expressions and returns the union of both
geometries.
