# GeoZarr-spec 0.1

This document aims to provides a geospatial extension to the Zarr specification (v2). Zarr specifies a protocol and format used for storing Zarr arrays, while the present extension defines **conventions** and recommendations for storing **multidimensional georeferenced grid** of geospatial observations (including rasters). 

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

## Status

This specification is an early draft (v0.1) deveveloped in the frame of an European Space Agency (ESA) GSTP. Through the optional General Support Technology Programme (GSTP) ESA, Participating States and Industry work together to convert promising engineering concepts into a broad spectrum of useable products.

## License

(CC BY 4.0) : Content in this repository is licensed under a Creative Commons Attribution 4.0 International  license. Licensees may copy, distribute, display, perform and make derivative works and remixes based on it only if they give the author or licensor the credits (attribution). You can find the complete text of this license at http://creativecommons.org/licenses/by/4.0/.

GeoZarr documentation by Christophe Noël (Spacebel) and other contributors.

## GeoZarr Classes

GeoZarr is restricted to geospatial data which conforms to the conceptual class of Dataset as defined below.

### GeoZarr DataArray

A GeoZarr DataArray variable is a **Zarr Array** that provides values of a measured or observed **geospatial phenomena** (possibly indirectly computed using processing). For example, it might provide reflectance values of a captured satellite scene, or it may describe average vegetation index (NDVI) values for defined period of times.

GeoZarr DataArray variable MUST include the attribute **\_ARRAY_DIMENSIONS which list the dimension names** (this property was first introduced by xarray library).


```
"_ARRAY_DIMENSIONS": [
        "lat",
        "lon"
    ]
```    
    
### GeoZarr Coordinates

GeoZarr Coordinates variable is a one dimensional **Zarr Array** that **indexes a dimension** of a GeoZarr DataArray (e.g latitude, longitude, time, wavelength).

GeoZarr Coordinates variable MUST include the attribute **\_ARRAY_DIMENSIONS equal to the Zarr array name** (e.g. latitude for the latitude Zarr Array).

### GeoZarr Auxiliary Data

GeoZarr Auxiliary variable is empty, one dimensional or multidimensional **Zarr Array** providing auxiliary information.

GeoZarr Auxiliary variable MUST include the attribute **\_ARRAY_DIMENSIONS set as an empty array**.

### GeoZarr Dataset

GeoZarr Dataset is a root **Zarr Group** which contains a **set of DataArray variables** (observed data), **Coordinates variables**, Auxiliary variables, and optionally children Datasets (located in children Zarr Groups). Dataset MUST contain a **consistent** set of data for which the DataArray variables have aligned dimensions, and share the same coordinates grid using a common spatial projection.

If multiple Array Variables share heterogenous dimensions or coordinates, a primary homogeneous set of variables MUST be located at root level, and the other sets declared in children datasets.


## GeoZarr Metadata

GeoZarr Arrays and Coordinates Variables MUST include [Climate and Forecast (CF)](http://cfconventions.org/) attributes (in the .attrs ojbect). The variables MUST include at least:

* standard_name for all variables
* grid_mapping (coordinates reference system) for all array variables

### Standard Name

A CF **standard name** is an attribute which identifies the physical quantity of a variable ([table](https://cfconventions.org/Data/cf-standard-names/78/build/cf-standard-name-table.html)). 

The quantity may describe the observed phenomenon for:
* a DataArray variable (for example 'surface_bidirectional_reflectance' for optical sensor data)
* a Coordinate variable
* an Auxiliary variable

The following standard names are recommended to describe coordinates variables for dimensions of DataArrays:
* grid_latitude, grid_longitude (spatial coordinates as degrees)
* projection_x_coordinate, projection_y_coordinates (spatial coordinates as per projection)
* sensor_band_identifier (multisptrectal band identifier)
* radiation_wavelength (hyperspectral wave length)
* altitude
* time

### Coordinate Reference System

The **grid_mapping** variable defined by DataArray variable defines  the coordinate reference system (CRS) used for the horizontal spatial coordinate values. The grid_mapping value indicates the Auxliary variable that holds all the CF attribute describing the CRS. 

### Other Properties

All other CF conventions are recommended, in particular the attributes below:

* add_offset
* scale_factor
* units (as per [UDUNITS v2](https://www.unidata.ucar.edu/software/udunits/udunits-2.2.28/udunits2.html))

## Multiscales

A GeoZarr DataArray variable might provide downscales of the data. In such case, the DataArray MUST define a multiscales attribute that includes the following properties 

* Path MUST provides a relative path the Zarr group which holds the same DataArray variable (based on name)
* Resolution is required to infer the required zoom level
* Zoom levels should be provided from highest to lowest resolution
* Recommended scale strategy is a doubled resolution for each level.
* First level path MUST reference to itself or can be omitted.

```diff
(mandatory items in red, optional items in green)
-{
-  "multiscales": [
-    {
-      "name": "example",
-      "datasets": [
-        {"path": ".", "resolution": "0.02"},
-        {"path": "1", "resolution": "0.04"},
-        {"path": "2", "resolution": "0.08"}
-      ],
+      "type": "gaussian",
-    }
-  ]
-}
```

## Quicklook

GeoZarr Dataset and DataArrray can define a quicklook defined by a path for each colour. As illustrated for blue colour, the path can be specified using a 

```diff
(mandatory items in red, optional items in green)
-{
-  "quicklook": [
-    {
-      "r": "rgb/red"
-    },
-    {
-      "g": "rgb/green"
-    },
-    {
-      "b": "rgb/data['2']"
"
-    }
-  ]
-}
```


## Rechunking

GeoZarr DataArrray MUST specify the paths to the rechunked instance of the data. These duplicates of the data enables to optimize queries on specific dimensions to improve performances (e.g. for requesting time series).

The attribute rechunking list the path the the various instances of the data. The corresponding Zarr metadata provides along the rechunked array provides the chunk size and shape.

```diff
(mandatory items in red, optional items in green)
-{
-  "rechunking": [
-    {
-      "path": "rechunk1"
-    },
-    {
-      "path": "rechunk2"
-    }
-  ]
-}


## Use Cases

### Multispectral Data

If the optical sensor captures spectral bands for different resolution, it is RECOMMENDED to hold the highest resolution dataset in the root group, and provide the other resolutions in children groups.

The spectral band SHOULD be represented as a dimension (not as an array neither a group).


