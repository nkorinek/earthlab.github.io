---
layout: single
title: "Export Numpy Arrays to Geotiff Format Using Rasterio and Python"
excerpt: "You often create outputs in Python that you want to use in another tool like QGIS or ArcGIS. Learn how to export a numpy array created through a rasterio workflow in Python to spatial geotiff."
authors: ['Leah Wasser', 'Chris Holdgraf']
dateCreated: 2018-04-14
modified: 2020-02-15
category: [courses]
class-lesson: ['multispectral-remote-sensing-data-python-veg-indices']
permalink: /courses/use-data-open-source-python/multispectral-remote-sensing/landsat-in-Python/vegetation-indices-in-python/export-numpy-array-to-geotiff-in-python/
nav-title: 'Export Raster Data'
week: 5
course: "intermediate-earth-data-science-textbook"
sidebar:
  nav:
author_profile: false
comments: true
order: 2
topics:
  remote-sensing: ['naip']
  reproducible-science-and-programming: ["python"]
  spatial-data-and-gis: ['raster-data']
redirect_from:
  - "/courses/earth-analytics-python/multispectral-remote-sensing-in-python/export-numpy-array-to-geotiff-in-python/"
---

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

* Calculate NDVI using NAIP multispectral imagery in `Python`.
* Describe what a vegetation index is and how it is used with spectral remote sensing data.
* Export or write a raster to a `.tif` file from `Python`.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What You Need

You will need a computer with internet access to complete this lesson and the Cold Springs Fire
data.

{% include/data_subsets/course_earth_analytics/_data-cold-springs-fire.md %}
</div>

## About Vegetation Indices

A vegetation index is a single value that quantifies vegetation health or structure.
The math associated with calculating a vegetation index is derived from the physics
of light reflection and absorption across bands. For instance, it is known that
healthy vegetation reflects light strongly in the near infrared band and less strongly
in the visible portion of the spectrum. Thus, if you create a ratio between light
reflected in the near infrared and light reflected in the visible spectrum, it
will represent areas that potentially have healthy vegetation.


## Normalized Difference Vegetation Index (NDVI)

The Normalized Difference Vegetation Index (NDVI) is a quantitative index of
greenness ranging from 0-1 where 0 represents minimal or no greenness and 1
represents maximum greenness.

NDVI is often used for a quantitate proxy measure of vegetation health, cover
and phenology (life cycle stage) over large areas.

<figure>
 <a href="{{ site.url }}/images/earth-analytics/remote-sensing/nasa-earth-observatory-ndvi-diagram.jpg">
 <img src="{{ site.url }}/images/earth-analytics/remote-sensing/nasa-earth-observatory-ndvi-diagram.jpg" alt="NDVI image from NASA that shows reflectance."></a>
    <figcaption>NDVI is calculated from the visible and near-infrared light
    reflected by vegetation. Healthy vegetation (left) absorbs most of the
    visible light that hits it, and reflects a large portion of
    near-infrared light. Unhealthy or sparse vegetation (right) reflects more
    visible light and less near-infrared light. Source: NASA
    </figcaption>
</figure>

* <a href="http://earthobservatory.nasa.gov/Features/MeasuringVegetation/measuring_vegetation_2.php" target="_blank">
More on NDVI from NASA</a>

## Calculate NDVI in Python

Sometimes you can download already calculated NDVI data products from a data provider. 

However, in this case, you don't have a pre calculated NDVI product from NAIP data. You need to calculate NDVI using the NAIP imagery / reflectance data that you have downloaded from Earth Explorer.

### How to Derive the NDVI Vegetation Index From Multispectral Imagery

The normalized difference vegetation index (NDVI) uses a ratio between near infrared
and red light within the electromagnetic spectrum. To calculate NDVI you use the
following formula where NIR is near infrared light and
red represents red light. For your raster data, you will take the reflectance value
in the red and near infrared bands to calculate the index.

`(NIR - Red) / (NIR + Red)`

You can perform this calculation using matrix math with the `numpy` library.

To get started, load all of the required python libraries. 

{:.input}
```python
import os
import matplotlib.pyplot as plt
import numpy as np
import rasterio as rio
import geopandas as gpd
import earthpy as et
import earthpy.spatial as es
import earthpy.plot as ep

# Download data and set working directory
data = et.data.get_data('cold-springs-fire')
os.chdir(os.path.join(et.io.HOME, 'earth-analytics'))
```

{:.output}
    Downloading from https://ndownloader.figshare.com/files/10960109
    Extracted output to /root/earth-analytics/data/cold-springs-fire/.



To begin open some data and create an output that you wish to export to geotiff format. Below you calculate NDVD from NAIP data using the earthpy `normalized_diff` function.

{:.input}
```python
naip_data_path = os.path.join("data", "cold-springs-fire", 
                              "naip", "m_3910505_nw_13_1_20150919", 
                              "crop", "m_3910505_nw_13_1_20150919_crop.tif")

with rio.open(naip_data_path) as src:
    naip_data = src.read()

naip_ndvi = es.normalized_diff(naip_data[3], naip_data[0])
```

{:.input}
```python
ep.plot_bands(naip_ndvi, cmap='PiYG', scale=False,
              vmin=-1, vmax=1,
              title="NAIP Derived NDVI\n 19 September 2015 - Cold Springs Fire, Colorado")
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-earth-data-science-textbook/05-multi-spectral-remote-sensing-python/vegetation-indices/2018-10-04-exp01-export-raster-to-geotiff/2018-10-04-exp01-export-raster-to-geotiff_6_0.png" alt = "NDVI for the Cold Springs Fire site derived from NAIP data. You may want to export this data as a geotiff to share and use in other tools like QGIS.">
<figcaption>NDVI for the Cold Springs Fire site derived from NAIP data. You may want to export this data as a geotiff to share and use in other tools like QGIS.</figcaption>

</figure>




## Export a Numpy Array to a Raster Geotiff in Python

When you are done, you can export your NDVI raster data so you could use them in
QGIS or ArcGIS or share them with your colleagues. To do this you use the `rio.write()`
function.

Exporting a raster in python is a bit different from what you may have learned using another language like `R`. In python, you need to: 

1. Create a new raster object with all of the metadata needed to define it. This metadata includes:
   * the shape (rows and columns) of the object
   * the coordinate reference system (crs)
   * the type of file (you will export a geotiff (.tif) in this lesson
   * and the type of data being stored (integer, float, etc). 
   
Lucky for you, all of this information can be accessed from the original NAIP data that you imported into python using attribute calls like:

`.transform` and
`.crs`

To implement this, below you will create a rasterio object to grab the needed spatial attributes.
   

{:.input}
```python
naip_data_path = os.path.join("data", "cold-springs-fire", 
                              "naip", "m_3910505_nw_13_1_20150919", 
                              "crop", "m_3910505_nw_13_1_20150919_crop.tif")

with rio.open(naip_data_path) as src:
    naip_data = src.read()
    naip_meta = src.profile

naip_meta
```

{:.output}
{:.execute_result}



    {'driver': 'GTiff', 'dtype': 'int16', 'nodata': -32768.0, 'width': 4377, 'height': 2312, 'count': 4, 'crs': CRS.from_wkt('PROJCS["UTM Zone 13, Northern Hemisphere",GEOGCS["GRS 1980(IUGG, 1980)",DATUM["unknown",SPHEROID["GRS80",6378137,298.257222101],TOWGS84[0,0,0,0,0,0,0]],PRIMEM["Greenwich",0],UNIT["degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",-105],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]]]'), 'transform': Affine(1.0, 0.0, 457163.0,
           0.0, -1.0, 4426952.0), 'tiled': False, 'compress': 'lzw', 'interleave': 'band'}





{:.input}
```python
naip_transform = naip_meta["transform"]
naip_crs = naip_meta["crs"]

# View spatial attributes
naip_transform, naip_crs
```

{:.output}
{:.execute_result}



    (Affine(1.0, 0.0, 457163.0,
            0.0, -1.0, 4426952.0),
     CRS.from_wkt('PROJCS["UTM Zone 13, Northern Hemisphere",GEOGCS["GRS 1980(IUGG, 1980)",DATUM["unknown",SPHEROID["GRS80",6378137,298.257222101],TOWGS84[0,0,0,0,0,0,0]],PRIMEM["Greenwich",0],UNIT["degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",-105],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]]]'))





You can view the type of data stored within the ndvi array using `.dtype`.
Remember that the naip_ndvi object is a numpy array.

{:.input}
```python
type(naip_ndvi), naip_ndvi.dtype
```

{:.output}
{:.execute_result}



    (numpy.ndarray, dtype('float64'))





Use `rio.open()` to create a new blank raster 'template'. 
Then write the NDVI numpy array to to that template using `dst.write()`.

Note that when we write the data we need the following elements:

1. the driver or type of file that we want to write. 'Gtiff' is a geotiff format
2. dtype: the structure of the data that you are writing. We are writing floating point values (values with decimal places)
3. the heigth and width of the ndvi object (accessed using the .shape attribute)
4. the crs of the spatial object (accessed using the rasterio NAIP data)
5. the transform information (accessed using the rasterio NAIP data)

Finally you need to specify the name of the output file and the path to where it will be saved on your computer. 



## Export a Numpy Array to a Raster Geotiff Using the Spatial Profile or Metadata of Another Raster
You can use the naip_meta variable that you created above. This variable contains all of the spatial metadata for naip data.

In this case, the 

1. number of bands (we have only one band vs 4 in the color image) and
2. the data format (we have floating point numbers - numers with decimals - vs integers)

have changed. Update those values then write out the image.

{:.input}
```python
naip_meta
```

{:.output}
{:.execute_result}



    {'driver': 'GTiff', 'dtype': 'int16', 'nodata': -32768.0, 'width': 4377, 'height': 2312, 'count': 4, 'crs': CRS.from_wkt('PROJCS["UTM Zone 13, Northern Hemisphere",GEOGCS["GRS 1980(IUGG, 1980)",DATUM["unknown",SPHEROID["GRS80",6378137,298.257222101],TOWGS84[0,0,0,0,0,0,0]],PRIMEM["Greenwich",0],UNIT["degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",-105],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]]]'), 'transform': Affine(1.0, 0.0, 457163.0,
           0.0, -1.0, 4426952.0), 'tiled': False, 'compress': 'lzw', 'interleave': 'band'}





{:.input}
```python
# Change the count or number of bands from 4 to 1
naip_meta['count'] = 1

# Change the data type to float rather than integer
naip_meta['dtype'] = "float64"
naip_meta
```

{:.output}
{:.execute_result}



    {'driver': 'GTiff', 'dtype': 'float64', 'nodata': -32768.0, 'width': 4377, 'height': 2312, 'count': 1, 'crs': CRS.from_wkt('PROJCS["UTM Zone 13, Northern Hemisphere",GEOGCS["GRS 1980(IUGG, 1980)",DATUM["unknown",SPHEROID["GRS80",6378137,298.257222101],TOWGS84[0,0,0,0,0,0,0]],PRIMEM["Greenwich",0],UNIT["degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",-105],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]]]'), 'transform': Affine(1.0, 0.0, 457163.0,
           0.0, -1.0, 4426952.0), 'tiled': False, 'compress': 'lzw', 'interleave': 'band'}





Note below that when we write the raster, we use `**naip_meta`
The two ** tells Python to unpack all of the values in the naip_meta object to use as arguments when writing the geotiff file. We already updated the elements that we needed to above (count and dtype). So this naip_meta object is ready to be used for the NDVI raster. 

{:.input}
```python
naip_ndvi_outpath = os.path.join("data", "cold-springs-fire", 
                                 "outputs", "naip_ndvi.tif")

# Write your the ndvi raster object
with rio.open(naip_ndvi_outpath, 'w', **naip_meta) as dst:
    dst.write(naip_ndvi, 1)
```




<div class="notice--info" markdown="1">

## Additional Resources

* <a href="https://phenology.cr.usgs.gov/ndvi_foundation.php" target="_blank">USGS Remote Sensing Phenology</a>
* <a href="http://earthobservatory.nasa.gov/Features/MeasuringVegetation/measuring_vegetation_2.php" target="_blank">NASA Earth Observatory - Vegetation Indices</a>

</div>