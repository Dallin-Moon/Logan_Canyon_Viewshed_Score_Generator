This document will contain a list of changes made to the score calculator toolbox, their location and code snippets.

#1

Modified to allow input observer features to be polylines and polygons.

This function was added to perform the conversion from polyline or polygon to point. This needs to happen so that the viewshed analysis can be run.

~~~
def convertToPoints(shape, type):
        if type = "Polygon":
            lines = arcpy.management.PolygonToLine(shape, "Observer_Line")
        else lines = shape
        return arcpy.management.GeneratePointsAlongLines(lines, "Observer_Points", "Percentage", "", 10, "END_POINTS")
~~~

And the logic to call this function is found in the execute() function.

~~~
if type.shapeType in ["Polyline", "Polygon"]:
        observer_points = convertToPoints(observer_points, type.shapeType)
~~~

It was also necessary to change how the tool handles the polygon datasets slightly.

~~~
def toOriginalGeometry(self, points, shape):
        shape = self.spatialJoin(shape, points, "Scored_Original_Geometry", "JOIN_ONE_TO_MANY")
        return shape
~~~

The above function joins the score data to the appropriate line or polygon. This means that the final score table will be a bit more messy for now as each polygon will have multiple viewshed scores joined to it. It is still legible and can easily be understood with a summary statistics.

Here is the line that is used to call the function near the end of the execute() block.

~~~
        if type.shapeType in ["Polyline", "Polygon"]:
            self.toOriginalGeometry(observer_points, original_shape)
~~~


#2

Modified to accept either a DSM **OR** a DEM and NLCD data to estimate a dsm.

This was added to practice use of parameters and updating parameters based on other parameters. It also increases funcionality and allows users to move forward without finding a DSM.

~~~
        if estimate_dsm:
            elev = arcpy.sa.Raster(elevation_raster)
            nlcd = arcpy.sa.Raster(nlcd_raster)
            remap = RemapValue([
                [11, 0],
                [21, 5],
                [22, 7],
                [23, 10],
                [24, 12],
                [31, 0],
                [41, 25],
                [42, 30],
                [43, 27],
                [52, 2],
                [71, 1],
                [81, 1],
                [82, 2],
                [90, 1],
                [95, 2]
            ])
            height_raster = Reclassify(nlcd, "Value", remap)
            dsm = elev + height_raster
            output_path = arcpy.env.workspace + "/estimated_dsm"
            dsm.save(output_path)
            elevation_raster = output_path
~~~

The parameter logic was updated to include the following two inputs

~~~
estimate_dsm = arcpy.Parameter(
            displayName="Estimate DSM?",
            name="estimate_dsm",
            datatype="GPBoolean",
            parameterType="Required",
            direction="Input"
            description="Uncheck this if you have a digital surface model. Leave checked and include land cover data to estimate a surface model with a DEM and land cover data."
        )
        estimate_dsm.value = True  # default

        nlcd_raster = arcpy.Parameter(
            displayName="NLCD Raster",
            name="nlcd_raster",
            datatype="GPRasterLayer",
            parameterType="Optional",
            direction="Input"
        )
~~~
And the following updateParameters() logic
~~~
    def updateParameters(self, parameters):
        """Modify the values and properties of parameters before internal
        validation is performed.  This method is called whenever a parameter
        has been changed."""
        estimate = parameters[1]
        nlcd_param = parameters[3]

        if estimate.value:
            nlcd_param.enabled = True
            nlcd_param.parameterType = "Required"
        else:
            nlcd_param.enabled = False
            nlcd_param.parameterType = "Optional"
            nlcd_param.value = None        # clear value when disabled

        return
~~~
