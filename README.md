# GEOM4009 MODIS Project
import os
from osgeo import ogr, gdal

os.chdir('/home/user/wrkdir/Group_Project/Data')

InputImage = 'MODIStest.tif'
Shapefile = 'AdmiraltyInletOutline.shp'
RasterFormat = 'GTiff'
PixelRes = 250
VectorFormat = 'ESRI Shapefile'

# Open datasets
Raster = gdal.Open(InputImage, gdal.GA_ReadOnly)
Projection = "+proj=lcc +lat_1=49 +lat_2=77 +lat_0=40 +lon_0=-100 +x_0=0 +y_0=0 +ellps=clrk66 +units=m +no_defs"

VectorDriver = ogr.GetDriverByName(VectorFormat)
VectorDataset = VectorDriver.Open(Shapefile, 0) # 0=Read-only, 1=Read-Write
layer = VectorDataset.GetLayer()
FeatureCount = layer.GetFeatureCount()
print("Feature Count:",FeatureCount)

# Iterate through the shapefile features
Count = 0
for feature in layer:
    Count += 1
    print("Processing feature "+str(Count)+" of "+str(FeatureCount)+"...")

    geom = feature.GetGeometryRef() 
    geom
    minX, maxX, minY, maxY = geom.GetEnvelope() # Get bounding box of the shapefile feature

    # Create raster
    OutTileName = str(Count)+'Image.Clip.tif'
    OutTile = gdal.Warp(OutTileName, Raster, format=RasterFormat, outputBounds=[minX, minY, maxX, maxY], xRes=PixelRes, yRes=PixelRes, dstSRS=Projection, resampleAlg=gdal.GRA_NearestNeighbour, options=['COMPRESS=DEFLATE'])
    OutTile = None # Close dataset

# Close datasets
Raster = None
VectorDataset.Destroy()
print("Done.")
