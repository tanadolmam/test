# MapHistPlot
Plot Heatmap from latitude and longtitude using Python. Then make it work with Tile Map Service.

## Overview
  To draw a heatmap, it requires several programs. I divide them it to 4 parts:
  
  [1. Clean data](#1-clean-data)
  
  [2. Divide data into small pieces](#2-export-rows-to-csv-file)
  
  [3. Process each piece](#3-plot-data-in-to-image-files)
  
  [4. Arrange the output](#4-change-format-to-tms)
  
## How to use
Firstly, install the following modules:
[PyMySQL](https://github.com/PyMySQL/PyMySQL),
[NumPy](https://github.com/numpy/numpy),
[Matplotlib](https://github.com/matplotlib/matplotlib) and 
[Pillow](https://github.com/python-pillow/Pillow)



### Window
1. Open Command Prompt
2. Change directory to file location  `cd C:\Users\user\Documents\GitHub`

3. Run each file in order
Example of plotting heatmap from zoomRange 6 to 12

`python p1-importToDB.py `

`python p2-splitCSV.py 12`

`python p3-plotHist2d.py 6 12`

`python p4-SXYtoTMS.py 6 12`




## Documentation

### p1-ImportCSVToDB.py
 
  Import raw data in form of .csv file to database. The format of data is shown in table below.
  
ID | latitude | longtitude | speed | date | direction | xx 
--- | --- | --- | --- | --- | --- | ---
int | float | float | float | date | int | int


An example of source file is in the Example folder.

We run function in the following order:
```
p1CreateCSVImport()
```
Create table name "CSVImport" in "test2" database.

```
p2ImportToTable(filePath)
```
* `filePath:string` - location of raw CSV file

Import .csv file from `filePath` to CSVImport table


```
p3CreateTemp()
```
Create table name "temp" in "test2" database to store data after cleaning. The primary keys are `lat` and `lon`


```
getTotalRows()
```
Return total count of a table "CSVImport" in database "test2"


```
p4UpsertTemp(chunkSize,totalRows)
```
* `chunkSize:int` - limit of reading rows
* `totalRows:int` - total rows of raw CSV file

Insert rows from "CSVImport to "temp". The duplicate primary key will be recalculate to a new row.




### p2-splitCSV.py
   Exports data from database into CSV file.
  
```
splitCSV(zoomRange,tile)
```

Read table "temp" from database name "test2" then export data in given `zoomRange` and `tile`(x,y). Tile must be in [Google Map tile](http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/) format.
  
The output will be stored in "/output/zoomZ/dataZ", where Z is `zoomRange`. For example, if `zoomRange` = 10, CSV file will be stored in "/output/zoom10/data10".
  
  
### p3-plotHist2D.py
   Plot data into 512x512 PNG files. The format can be use in Google Map Overlay an Longdo Map Layer.
  
```
generateCmap()
```
Return hot array colormap but first element(minimum value) will be transparent. 

```
plotting(zoomRange,xmin,xmax,ymin,ymax,colorMap)
```
* `zoomRange:int` - Range of zoom
* `xmin:int` - most left value of x axis in map tile system
* `xmax:int` - most right value of x axiss in map tile system
* `ymin:int` - top value of y axis in map tile system
* `ymax:int` - bottom value of y axis in map tile system
* `colorMap:list` - list of 4-tuples in format of (R,G,B,A)

Read CSV file from "output/zoomZ/dataZ" and store in 2 lists, __latitude__ and __longtitude__. This function will append point to a list equal to __speed__ of that point
For example, if (lat,lon,spd) = (10.3,15.2,120). The __latitude__ list will have 10.2 equal to 120 elements.
After this function read 500,000 rows of data or read to the end of csv soruce file, it will call `createHist2d(...)`

```
createHist2d(lonList,latList, binSize,imgName,lt,rb,cm)
```
* `latList:float` - array of latitude
* `lonList:float` - array of lontitude
* `binSize:int` - matplotlib.pyplot.hist2d bins
* `imgName:string` - output file name(need .png at the end)
* `lt:(float,float)` - (lat,lon) of left top point of a bound
* `rb:(float,float)` - (lat,lon) of bottom right point of a bound
* `cm:list` - list of 4-tuples in format of (R,G,B,A)

Draw 2D histogram on canvas size equal to `lt` and `rb` bound and save to "output/zoomZ/tempZ".
  
```
cropImage(zoomRange,xmin,xmax,ymin,ymax)
```
* `zoomRange:int` - Range of zoom
* `xmin:int` - most left value of x axis in map tile system
* `xmax:int` - most right value of x axiss in map tile system
* `ymin:int` - top value of y axis in map tile system
* `ymax:int` - bottom value of y axis in map tile system

Create template images for cropping and trim all images in `zoomRange` using bounding box range from `getRange()`.

```
getRange(zoomRange,x,y)
```
* `zoomRange:int` - Range of zoom
* `x:int` - x coordinate for reference tile (template image)
* `y:int` - y coordinate for reference tile (template image)

Return ranges of x axis and y axis in pixel .These ranges will be used for cropping other images.

```
stitchTile(zoomRange)
```
* `zoomRange:int` - Range of zoom

Create a new tile in given `zoomRange` by stitching 4 tiles(images) from `zoomRange+1` as one.


### p4-XYZtoTMS.py
   Change file's name and organize them in Tile Map Service format.
  
```
toTMS(zoomRange,xmin,xmax,ymin,ymax)
```
* `zoomRange:int` - Range of zoom
* `xmin:int` - most left value of x axis in map tile system
* `xmax:int` - most right value of x axiss in map tile system
* `ymin:int` - top value of y axis in map tile system
* `ymax:int` - bottom value of y axis in map tile system

Organize files to make them work with Longdo Map API(TMS).


### Extra
  These files are used as library, they locate in lib folder.
  1. pathCheck.py
```
createFolder(filePath)
```
* `filePath:string` - Location for new folder

Check for `filePath`, if not exist, create it.
 
 
 2. mapTool.py
```
tile2long(x,z) 
```
* `x:int` - x coordinate of a tile
* `z:int` - zoomRange

Return a minimum longtitude of all tiles in `x` axis

```
tile2lat(y,z) 
```
* `y:int` - y coordinate of a tile
* `z:int` - zoomRange

Return a minimum latitude of all tiles in `y` axis

```
getTileBound(zoomRange) 
```
* `zoomRange:int` - Range of zoom

Return bouding coordinate (x,y) of all tiles those cover Thailand in format of 5 parameters (`zoomRange`,`xmin`,`xmax`,`ymin`,`ymax`)

```
bgColor(zoomRange,opacity) 
```
* `zoomRange:int` - Range of zoom
* `opacity:int` - opacity of background (integer between 0-255), default=130

Fill image background with black.

```
retouch(zoomRange,brightness,contrast,color,sharpness) 
```
* `zoomRange:int` - Range of zoom
* `brightness:int` - configuration brightness of an image
* `contrast:int` - configuration contrast of an image
* `color:int` - configuration color of an image
* `sharpness:int` - configuration sharpness of an image

More information about [ImageEnhance](https://pillow.readthedocs.io/en/3.0.x/reference/ImageEnhance.html)

## References

<https://www.strava.com/heatmap>

<http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/>

<https://medium.com/strava-engineering/the-global-heatmap-now-6x-hotter-23fc01d301de>




