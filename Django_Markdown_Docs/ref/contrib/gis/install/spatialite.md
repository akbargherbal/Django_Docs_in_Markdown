# Installing SpatiaLite

[SpatiaLite](https://www.gaia-gis.it/fossil/libspatialite/index) adds spatial support to SQLite, turning it into a full-featured
spatial database.

First, check if you can install SpatiaLite from system packages or binaries.

For example, on Debian-based distributions that package SpatiaLite 4.3+, try to
install the `libsqlite3-mod-spatialite` package. For older releases install
`spatialite-bin`.

For macOS, follow the [instructions below](#spatialite-macos).

For Windows, you may find binaries on the [Gaia-SINS](https://www.gaia-gis.it/gaia-sins/) home page.

In any case, you should always be able to [install from source](#spatialite-source).

<a id="spatialite-source"></a>

## Installing from source

[GEOS and PROJ](geolibs.md) should be installed
prior to building SpatiaLite.

### SQLite

Check first if SQLite is compiled with the [R\*Tree module](https://www.sqlite.org/rtree.html). Run the sqlite3
command line interface and enter the following query:

```sqlite3
sqlite> CREATE VIRTUAL TABLE testrtree USING rtree(id,minX,maxX,minY,maxY);
```

If you obtain an error, you will have to recompile SQLite from source. Otherwise,
skip this section.

To install from sources, download the latest amalgamation source archive from
the [SQLite download page](https://www.sqlite.org/download.html), and extract:

```shell
$ wget https://www.sqlite.org/YYYY/sqlite-amalgamation-XXX0000.zip
$ unzip sqlite-amalgamation-XXX0000.zip
$ cd sqlite-amalgamation-XXX0000
```

Next, run the `configure` script – however the `CFLAGS` environment variable
needs to be customized so that SQLite knows to build the R\*Tree module:

```shell
$ CFLAGS="-DSQLITE_ENABLE_RTREE=1" ./configure
$ make
$ sudo make install
$ cd ..
```

<a id="spatialitebuild"></a>

### SpatiaLite library (`libspatialite`)

Get the latest SpatiaLite library source bundle from the
[download page](https://www.gaia-gis.it/gaia-sins/libspatialite-sources/):

```shell
$ wget https://www.gaia-gis.it/gaia-sins/libspatialite-sources/libspatialite-X.Y.Z.tar.gz
$ tar xaf libspatialite-X.Y.Z.tar.gz
$ cd libspatialite-X.Y.Z
$ ./configure
$ make
$ sudo make install
```

#### NOTE
For macOS users building from source, the SpatiaLite library *and* tools
need to have their `target` configured:

```shell
$ ./configure --target=macosx
```

<a id="spatialite-macos"></a>

## macOS-specific instructions

To install the SpatiaLite library and tools, macOS users can use [Homebrew](https://brew.sh/).

### Homebrew

[Homebrew](https://brew.sh/) handles all the SpatiaLite related packages on your behalf,
including SQLite, SpatiaLite, PROJ, and GEOS. Install them like this:

```shell
$ brew update
$ brew install spatialite-tools
$ brew install gdal
```

Finally, for GeoDjango to be able to find the SpatiaLite library, set
the `SPATIALITE_LIBRARY_PATH` setting to its path. This will be within
your brew install path, which you can check with:

```console
$ brew --prefix
/opt/homebrew
```

Using this brew install path, the full path can be constructed like this:

```default
SPATIALITE_LIBRARY_PATH = "/opt/homebrew/lib/mod_spatialite.dylib"
```
