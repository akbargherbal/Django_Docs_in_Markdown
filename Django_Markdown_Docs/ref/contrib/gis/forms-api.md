# GeoDjango Forms API

GeoDjango provides some specialized form fields and widgets in order to visually
display and edit geolocalized data on a map. By default, they use
[OpenLayers](https://openlayers.org/)-powered maps, with a base WMS layer provided by [NASA](https://www.earthdata.nasa.gov/).

## Field arguments

In addition to the regular [form field arguments](../../forms/fields.md#core-field-arguments),
GeoDjango form fields take the following optional arguments.

### `srid`

#### Field.srid

This is the SRID code that the field value should be transformed to. For
example, if the map widget SRID is different from the SRID more generally
used by your application or database, the field will automatically convert
input values into that SRID.

### `geom_type`

#### Field.geom_type

You generally shouldn’t have to set or change that attribute which should
be set up depending on the field class. It matches the OpenGIS standard
geometry name.

## Form field classes

### `GeometryField`

### *class* GeometryField

### `PointField`

### *class* PointField

### `LineStringField`

### *class* LineStringField

### `PolygonField`

### *class* PolygonField

### `MultiPointField`

### *class* MultiPointField

### `MultiLineStringField`

### *class* MultiLineStringField

### `MultiPolygonField`

### *class* MultiPolygonField

### `GeometryCollectionField`

### *class* GeometryCollectionField

## Form widgets

GeoDjango form widgets allow you to display and edit geographic data on a
visual map.
Note that none of the currently available widgets supports 3D geometries, hence
geometry fields will fallback using a `Textarea` widget for such data.

### Widget attributes

GeoDjango widgets are template-based, so their attributes are mostly different
from other Django widget attributes.

#### BaseGeometryWidget.geom_type

The OpenGIS geometry type, generally set by the form field.

#### BaseGeometryWidget.map_srid

SRID code used by the map (default is 4326).

#### BaseGeometryWidget.display_raw

Boolean value specifying if a textarea input showing the serialized
representation of the current geometry is visible, mainly for debugging
purposes (default is `False`).

#### BaseGeometryWidget.supports_3d

Indicates if the widget supports edition of 3D data (default is `False`).

#### BaseGeometryWidget.template_name

The template used to render the map widget.

You can pass widget attributes in the same manner that for any other Django
widget. For example:

```default
from django.contrib.gis import forms


class MyGeoForm(forms.Form):
    point = forms.PointField(widget=forms.OSMWidget(attrs={"display_raw": True}))
```

### Widget classes

`BaseGeometryWidget`

### *class* BaseGeometryWidget

This is an abstract base widget containing the logic needed by subclasses.
You cannot directly use this widget for a geometry field.
Note that the rendering of GeoDjango widgets is based on a template,
identified by the [`template_name`](#django.contrib.gis.forms.widgets.BaseGeometryWidget.template_name) class attribute.

`OpenLayersWidget`

### *class* OpenLayersWidget

This is the default widget used by all GeoDjango form fields.
`template_name` is `gis/openlayers.html`.

`OpenLayersWidget` and [`OSMWidget`](#django.contrib.gis.forms.widgets.OSMWidget) use the `ol.js` file hosted
on the `cdn.jsdelivr.net` content-delivery network. You can subclass
these widgets in order to specify your own version of the `ol.js` file in
the `js` property of the inner `Media` class (see
[Assets as a static definition](../../../topics/forms/media.md#assets-as-a-static-definition)).

`OSMWidget`

### *class* OSMWidget

This widget uses an OpenStreetMap base layer to display geographic objects
on. Attributes are:

#### template_name

`gis/openlayers-osm.html`

#### default_lat

#### default_lon

The default center latitude and longitude are `47` and `5`,
respectively, which is a location in eastern France.

#### default_zoom

The default map zoom is `12`.

The [`OpenLayersWidget`](#django.contrib.gis.forms.widgets.OpenLayersWidget) note about JavaScript file hosting above also
applies here. See also this [FAQ answer](https://help.openstreetmap.org/questions/10920/how-to-embed-a-map-in-my-https-site) about `https` access to map
tiles.
