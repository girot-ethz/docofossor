# Documentation

## Docofossor list format

The Docofossor format is used to calculate boolean operations within the 2.5D distance field. Docofossor's data structure is based on a single list that defines a regular spaced quad grid from topographic data. The Docofossor list `df[]` consist of a header part (dimension list) that defines the properties of the grid such as the cell size, the number of rows and columns, and the coordinates of the origin of the grid. The header information is followed by z-values coming from a [Digital Terrain Model (DTM)](https://en.wikipedia.org/wiki/Digital_elevation_model) in column-major order starting bottom left. 

## Dimension list

The first 10 values of the `df[]` list holds the following information, specified as the `dim[]` list:

|Line|type|Variable|Description|
|-|-|-|-|
|dim[0]|int|nc|Number of columns|
|dim[1]|int|nr|Number of rows|
|dim[2]|float|ox|X coordinate of the local origin (lower left corner of the grid, center of the cell)|
|dim[3]|float|oy|Y coordinate of the local origin (lower left corner of the grid, center of the cell)|
|dim[4]|float|cx|Cellsize X|
|dim[5]|float|cy|Cellsize Y|
|dim[6]||||
|dim[7]||||
|dim[8]|float|gx|X coordinate of the global origin (lower left corner of the grid, center of the cell)|
|dim[9]|float|gy|Y coordinate of the global origin (lower left corner of the grid, center of the cell)|
>Note: Position 6 and 7 are left free to future-proof the `df[]` list.

## List of z-values

The values after line 10 of the `df[]` list hold the z-values of the digital terrain model, specified as the `lz[]` list.

|Line|Type|Description|
|-|-|-|
|lz[0]|float|Z-value of the lower left corner of the grid
|lz[1]|float|Next Z-value of the grid in column-major order
|lz[2]|float|Next Z-value of the grid in column-major order
|...|||
|lz[n]|float|Z-value of the upper right corner of the grid

## Components

Docofossor is using IronPython within Rhino Grasshopper to make the calculations. The components are separated in categories in the Grasshopper toolbar. For now, there are five categories comprising *Grid*, *Relative Operations*, *Absolute Operations*, *Analysis*, and *Geometry*. They are made available as a set of open source UserObjects for Grasshopper. Use the list below to jump to the respective component descriptions.

* __I/O__
  * [dfEmptyGrid](#-dfemptygrid)
  * [dfImportDF](#-dfimportdf)
  * [dfImportPoints](#-dfimportpoints)
  * [dfImportXYZ](#-dfimportxyz)
  * [dfExportDF](#-dfexportasc)
  * [dfExportXYZ](#-dfexportxyz)
  
* __Grid__
  * [dfGridShift](#-dfgridshift)
  * [dfGridGlobal](#-dfgridglobal)
  * [dfGridCompare](#-dfgridcompare)
  * [dfGridInfo](#-dfgridinfo)
  * [dfGridAddition](#-dfgridaddition)
  * [dfGridFilter](#-dfgridfilter)
  * [dfGridInterpolation](#-dfgridinterpolation)
  * [dfGridRegion](#-dfgridregion)
  * [dfGridSmooth](#-dfgridsmooth)

* __Operations Relative__
  * [dfCutOnPoint](#-dfcutonpoint)
  * [dfFillOnPoint](#-dffillonpoint)
  * [dfCutOnPath](#-dfcutonpath)
  * [dfFillOnPath](#-dffillonpath)
  * [dfCutOnArea](#-dfcutonarea)
  * [dfFillOnArea](#-dffillonarea)

* __Operations Absolute__
  * [dfCutInPoint](#-dfcutinpoint)
  * [dfFillInPoint](#-dffillinpoint)
  * [dfCutFillInPath](#-dfcutfillinpath) 
  * [dfCutInPath](#-dfcutinpath)
  * [dfFillInPath](#-dffillinpath) 
  * [dfCutFillInSurface](#dfcutfillinsurface)
  * [dfCutInSurface](#-dfcutinsurface)
  * [dfFillInSurface](#-dffillinsurface) 

* __Generative__
  * [dfNoise](#-dfnoise)
  * [dfSineWave](#-dfsinewave)

* __Analysis__
  * [dfShortestPath](#-dfshortestpath)
  * [dfSlopeVector](#-dfslopevector)
  * [dfViewshed](#-dfviewshed)

* __Geometry__
  * [dfGridMesh](#-dfgridmesh)
  * [dfGridPoints](#-dfgridmesh)

### I/O

#### ![img](/img/dfEmptyGrid.png "jh") dfEmptyGrid
Creates an empty grid of Z-values and returns the list and the dimensions

|Inputs|Description|
|-|-|
|nc|number of columns
|nr|number of rows
|ox|offset in X
|oy|offset in Y
|cx|cellsize X
|cy|cellsize Y
|__Output__||
|df|The Docofossor list of grid-dimensions and Z-values

#### ![img](/img/dfImportDF.png "jh") dfImportDF

Reads the distance field from a Docofossor data file (.df).

|Inputs|Description|
|-|-|
|f|The filepath to the df-file
|n|Number of rows and columns to skip (every n-th r/c)
|__Output__||
|df|The Docofossor list

#### ![img](/img/dfImportPoints.png "jh") dfImportPoints

Creates the Docofossor distance field from points on a regular grid.

|Inputs|Description|
|-|-|
|pts|Points to convert to Docofossor distance field (only ortogonal and regular grid points)
|__Output__||
|df|The Docofossor list

#### ![img](/img/dfImportXYZ.png "jh") dfImportXYZ

The *Import XZY* component imports a text file to a *df* list that has topographic data stored as a list of x, y, and z values separated by whitespace characters. Each point should start at a new line.

|Input|Description|
|--|--|
|f|The filepath to the xyz-file (it will also take txt files)
|n|Number of rows and columns to skip (every n-th r/c)
|sx|Translates the grid to a local X-origin. The original origin is stored and used to restore the grid to global coordinates at export time.
|sy|Translates the grid to a local Y-origin. The original origin is stored and used to restore the grid to global coordinates at export time.
|__Output__||
|df|The Docofossor list

#### ![img](/img/dfExportDF.png) dfExportDF
Writes a new .df file containing the dimensions and z-values of the docofossor list in global coordinates.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|f|The name of the file
|w|Use a boolean button, set to true to start writing

#### ![img](/img/dfExportXYZ.png) dfExportXYZ
Writes a new XYZ file of the point locations in global coordinates.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|f|The name of the file
|w|Use a boolean button, set to true to start writing

### Grid

#### ![img](/img/dfGridGlobal.png) dfGridGlobal
Sets the Docofossor grid back to the original global coordinates

|Inputs|Description|
|-|-|
|df|The Docofossor list to work on
|__Output__||
|df|The Docofossor list in global coordinates

#### ![img](/img/dfGridShift.png) dfGridShift
Shifts the Docofossor grid in local coordinates. If left blank the grid origin will be set to x=0 and y=0. The original origin is stored and used to restore the grid to global coordinates at export time.

|Inputs|Description|
|-|-|
|df|The Docofossor list to work on
|sx|Shift the origin of the grid in X direction
|sy|Shift the origin of the grid in Y direction
|__Output__||
|df|The Docofossor list with the new origin

#### ![img](/img/icons/dfGridCompare.png) dfGridCompare
Compares two terrains (lists of Z-values) with each other.

|Inputs|Description|
|-|-|
|df1|The first Docofossor list
|df2|The second Docofossor list to compare with
|__Output__||
|cut|The volume added from df1 to df2
|fill|The volume added from df1 to df2
|balance|The volume difference
|df|Docofossor list with difference values (delta landscape)

#### ![img](/img/dfGridInfo.png) dfGridInfo

Outputs grid information data for reference

|Inputs|Description|
|-|-|
|df|The Docofossor list
|__Output__||
|info|Grid information data

#### ![img](/img/dfGridAddition.png) dfGridAddition
Adds two Docofossor distance fields together

|Inputs|Description|
|-|-|
|df|The original Dofofossor distance field
|df_add|The Docofossor distance field to add
|__Output__||
|df|The new Docofossor distance field

#### ![img](/img/dfGridFilter.png) dfGridFilter
Filters a list of Z-values to include only every n-th row and column

|Inputs|Description|
|-|-|
|df|The Docofossor list to work on
|n|Number of rows and columns to skip (every n-th r/c)
|__Output__||
|df|The new Docofossor list of grid-dimensions and Z-values

#### ![img](/img/dfGridInterpolation.png) dfGridInterpolation
Increases the resolution of the Docofossor distance field by 2, using linear interpolation

|Inputs|Description|
|-|-|
|df|The Docofossor distance field
|__Output__||
|df|The Docofossor distance field in 2x resolution

#### ![img](/img/dfGridRegion.png) dfGridRegion
Crops the grid to a curve (bounding box), for faster operation

|Inputs|Description|
|-|-|
|df|Docofossor list of grid-dimensions and Z-values to work on
|crv|The curve to crop the grid
|__Output__||
|df|The new Docofossor list of grid-dimensions and Z-values

#### ![img](/img/icons/dfGridSmooth.png) dfGridSmooth
Smoothens a landscape by applying a 2D Gaussian convolution kernel.

|Inputs|Description|
|-|-|
|df|Docofossor list of grid-dimensions and Z-values to work on
|rad|The radius of the kernel (5 if not specified)
|__Output__||
|df|The new Docofossor list of grid-dimensions and z-values


        
        

### Operations Relative

### Operations Absolute

### Generative

### Analysis

### Geometry
















*************************************************************************************************
























### Relative Operations

#### ![img](/img/icons/dfCutOnPoint.png) dfCutOnPoint
Creates an excavation on a point as a sine curve.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|pt|Excavation location (point3D)
|mxd|Maximum depth
|sa|Slope angle
|method|Boolean toggle between sine-deposition and relative-deposition
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutOnPath.png) dfCutOnPath
Creates an excavation along a path curve.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|crv|The path curve
|wt|Width at the top of the carving tool
|wb|Width at the bottom of the carving tool
|d|Maximum depth at the center
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutOnArea.png) dfCutOnArea
> coming soon

#### ![img](/img/icons/dfFillOnPoint.png) dfFillOnPoint
Creates a deposition on a point as a sine curve (relative).

|Inputs|Description|
|-|-|
|df|Docofossor list of grid-dimensions and Z-values to work on
|pt|Deposition location (point3D)
|mxh|Maximum height
|sa|Slope angle
|method|Boolean toggle between sine-deposition and relative-deposition
|__Output__||
|df|The Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfFillOnPath.png) dfFillOnPath
> coming soon

#### ![img](/img/icons/dfFillOnArea.png) dfFillOnArea
Creates a deposition / mound within a boundary curve

|Inputs|Description|
|-|-|
|df|Docofossor list of Z-values to work on
|crv|The boundary curve
|mxh|Maximum height
|sa|Slope angle
|__Output__||
|df|The Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutVolumeOnArea.png) dfCutVolumeOnArea
>coming soon

#### ![img](/img/icons/dfFillVolumeOnArea.png) dfFillVolumeOnArea
Creates a volume deposition within a boundary curve.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|crv|The boundary curve
|mxh|Maximum height
|sa|Slope angle
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

### Absolute Operations

#### ![img](/img/icons/dfCutInPoint.png) dfCutInPoint
Creates an excavation in a point as a sine curve (absolute).

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|pt|Excavation location (point3D)
|sa|Slope angle
|method|Boolean toggle between sine-deposition and relative-sine-deposition
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutInPath.png) dfCutInPath

Creates a rectangular carving along a path curve.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|crv|The path curve
|w|Total width of the carving tool
|d|Maximum depth at the center
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutInPaths.png) dfCutInPaths

Creates a trapezoidal carving (slope angle) along several path curves.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|crv|A list of path curves
|sa|Slope angle
|wb|Width at the bottom of the carving tool
|__Output__||
|df|Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfCutInSurface.png) dfCutInSurface
>coming soon

#### ![img](/img/icons/dfFillInPoint.png) dfFillInPoint
Creates a deposition in a point as a sine curve (absolute).

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|pt|Deposition location (point3D)
|sa|Slope angle
|method|Boolean toggle between sine-deposition and relative-sine-deposition
|__Output__||
|df|The Docofossor list of grid-dimensions and Z-values
|vol|The volume delta

#### ![img](/img/icons/dfFillInPath.png) dfFillInPath
> coming soon

#### ![img](/img/icons/dfFillInSurface.png) dfFillInSurface
> coming soon

#### ![img](/img/icons/dfCutFillInPath.png) dfCutFillInPath
> coming soon

#### ![img](/img/icons/dfCutFillInSurface.png) dfCutFillInSurface

Fits the landscape to a given surface by pulling the points (both cut and fill) and connects to the surrounding with a slope.

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|srf|The surface to drag the points to
|sa|Slope angle
|__Output__||
|df|The new list of grid-dimensions and Z-values
|vol|The volume delta

### Analysis



#### ![img](/img/icons/dfShortestPath.png) dfShortestPath

Calculates the shortest path between to points.

|Inputs|Description|
|-|-|
|df|the Docofossor distance field
|omap|list of obstacles, 1 is free, 0 is occupied
|sp|the starting point
|tp|the target point
|c|type of neighborhood, allowed moves. 1 > 4 neighbors sharing an edge. 2 > 8 neighbors sharing a vertex
|f|factor to multiply height difference (0=no influence)
|__Output__||
|dsts|list of distance values for each cell to the target.
|route|a list of tuples along the path, format: (distance, ix, iy)

#### ![img](/img/icons/dfSlopeGradientVector.png) dfSlopeGradientVector
Calculates the gradient direction vectors.

|Inputs|Description|
|-|-|
|df|Docofossor list of grid-dimensions and z-values to work on
|__Output__||
|a|List of gradient vectors, magnitude corresponding to slope

#### ![img](/img/icons/dfViewshed.png) dfViewshed
Analyses the visibilty (3d viewshed) from a given start point.

|Inputs|Description|
|-|-|
|df|Docofossor list of grid-dimensions and Z-values to work on
|pt|The position of the viewer to be analysed.
|h|Height of the eye above ground. Optional, 1.6 if omitted.
|__Output__||
|va|List of visibilities (Boolean) for each point.
|spt|Point object indicating the actual position used for calculation (debugging).

#### ![img](/img/icons/dfHeightmapShader.png) dfHeightmapShader
Creates a Mesh texture gradient where the lowest faces are colored black, and the highest are colored white.

|Inputs|Description|
|-|-|
|m|The mesh coming from the dfGridMesh component.
> For now, this component contains a Cluster of native Grasshopper components. Double click to adjust the parameters within the Cluster. This component will be rewritten in Python to take advantage of the Docofossor distance field.

#### ![img](/img/icons/dfHillShader.png) dfHillShader
Creates a Mesh texture gradient that simulates face-exposure coming from the sun. Full exposure will be colored white, no exposure black.

|Inputs|Description|
|-|-|
|m|The mesh coming from the dfGridMesh component.
> For now, this component contains a Cluster of native Grasshopper components. Double click to adjust the parameters within the Cluster. This component will be rewritten in Python to take advantage of the Docofossor distance field.

#### ![img](/img/icons/dfSlopeShader.png) dfSlopeShader
Creates a Mesh texture where any mesh-face over 33% slope is colored red.

|Inputs|Description|
|-|-|
|m|The mesh coming from the dfGridMesh component.
> For now, this component contains a Cluster of native Grasshopper components. Double click to adjust the parameters within the Cluster. This component will be rewritten in Python to take advantage of the Docofossor distance field.

### Geometry
#### ![img](/img/icons/dfGridMesh.png) dfGridMesh

Creates a mesh of quads on the point grid.

|Inputs|Description|
|-|-|
|df|Docofossor list of Z-values to work on
|__Output__||
|m|The newly created mesh

#### ![img](/img/icons/dfGridPoints.png) dfGridPoints

Creates points from z values and grid dimensions

|Inputs|Description|
|-|-|
|df|Docofossor list to work on
|__Output__||
|pts|The points (x,y,z)