# Instructions for the Logan Canyon Viewshed Score Calculator Toolbox

## Overview

This tool takes a set of inputs:
    - Input Points (Cell towers, Lamp Posts, Signs)
    - Elevation Raster
    - Buffer Distance
    - Output GeoDatabase
    - List of Scoring Parameters

Using these inputs it will calculate the viewshed visible from each point individually. It will then convert those viewsheds to vector polygons and perform spatial joins on each viewshed individually to determine which scoring parameters are covered by the polygon. The parameters need a "score" attribute with a number. These scores will be summed and output as a final score in the input point attribute table.

## Data Preparation

Preparation for each input includes:
    - Adding a score field to each parameter attribute table. The score field must contain a number. Most features will have the same number for each parameter feature, but some may have different values. In the case of having multiple values, the polygon will retain the highest score from that feature class, otherwise it will retain a count of the number intersected features from that class, and a total score of the product of the score by the count. e.g. A road feature class with a score field of 2 would output 2 times the number of roads intersected by the polygon. Five roads covered by one polygon would result in a score of 10.
    - Buffer distance will be in the units of the projected or unprojected coordinate system of the map you are working in.
    - The resolution of the elevation raster will affect the output by increasing or decreasing generalization of the analysis.
    - Observer Points must have a field called "OFFSETA" denoting the height of the viewpoint relative to the elevation raster uesd. This will be in the units of the coordinate system of the observer point feature class.

It is best practice to project all inputs to the same projection so that all of the units are standard. Improvements will be made in the future to make the units of inputs dynamic. Documentation will be updated when improvements are made.
## Using the Tool

Simply input the appropriate features into the tool and let it run. Tool will run at different speeds depending on the machine. During testing on an Intel Core i7, 16gb RAM and 6gb GPU, ten input points ran in about 5 minutes. Increasing this number to 55 input points increased runtime to about 40 minutes.

Note: 
    - Tool outputs information in the 'Details' pane while running.
    - Bulk of analysis is the viewshed generation.
    - Final output will be just the polygon data, and the input points with an updated table.

## Understanding the Output

Output will appear in a table like this:

| ID | F-Type| OFFSETA  | Score  | Parameter 1       | Parameter 2       | Parameter 3       | Parameter 4        | Parameter 5       |
|----|-------|----------|--------|-------------------|-------------------|-------------------|--------------------|-------------------|
| 1  | Point | 20       | 15     | Score: 0 Count: 0 | Score: 3 Count: 1 | Score: 0 Count: 0 | Score: 10 Count: 2 | Score: 2 Count: 1 |
| 2  | Point | 20       | 10     | Score: 0 Count: 0 | Score: 3 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 2 Count: 1 |
| 3  | Point | 20       | 18     | Score: 0 Count: 0 | Score: 4 Count: 1 | Score: 0 Count: 0 | Score: 10 Count: 2 | Score: 4 Count: 2 |
| 4  | Point | 20       | 16     | Score: 3 Count: 1 | Score: 4 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 4 Count: 2 |
| 5  | Point | 20       | 15     | Score: 6 Count: 2 | Score: 2 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 2 Count: 1 |
| 6  | Point | 20       | 22     | Score: 6 Count: 2 | Score: 2 Count: 1 | Score: 0 Count: 0 | Score: 10 Count: 2 | Score: 4 Count: 2 |
| 7  | Point | 20       | 14     | Score: 3 Count: 1 | Score: 4 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 2 Count: 1 |

Here is the breakdown
    - OFFSETA is the height of the observer point used for the viewshed analysis
    - Score is the final score of that point. This isn't golf, so a higher score is better.
    - Each parameter follows this format:
        - Score: ## Count: ##
        - Score is the number attributed to the score by that parameter for that point
        - Count is the number of features found covered by the polygon generated from the point
        - Multi score features will always show a count of 1, and hold the highest scoring feature that was found intersecting the polygon
            - Parameter 2 is a mutli score feature. This is apparent because the scores aren't multiples of each other. In this example they range from 1-5 
        - The final score is the sum of the scores. The score divided by the count is the score of each individual parameter feature.
            - Parameter 4 is a score of 5 per feature intersected.

The ArcGIS Experience provided has an example 55 points placed along Logan Canyon. Specific points can be selected from the table provided there, then their individual scores can be summed to provide a score for all of the points combined. Other statistics are also provided there such as standard deviation and the mean score.

## Limitations to Consider

Limitations and uncertainty are always companions to geospatial projects. This tool attempts to minimize specific limitations that stem from the workflow itself. Other limitations can be introduced by the input data from the user. The tool attempts to limit the following factors:
    - Score calculations can be skewed by input parameters. Polylines can disproportionately attribute to the score if not dissolved properly. The tool currently dissolves all polylines, unless the polyline has a multi-value score field, to minimize this skew.
        - Say a polyline has ten segments that are only a few feet long, this will register in the tool as ten separate features and greatly increase the score of that viewshed. By dissolving the polylines entirely it will avoid this issue.
    - Uncertainty also stems from the DEM used to calculate viewsheds. These viewsheds are generalizations and many times result in multiple separate pieces. Some pieces are very small and close together, and can intersect the same features which would increase the score incorrectly. To avoid this the viewsheds are aggregated to a degree to combine the smaller pieces into one if deemed close enough to each other.
    - Score values of parameters are determined by users. Careful care must be taken to give each parameter a value that represents its impact on the target scenario appropriately.

## Data Credits

    - United States Geological Survey (USGS). (2022). Land Cover Data. Retrieved from https://www.sciencebase.gov/catalog/item.
    - United States Geological Survey (USGS). (2024). Digital Elevation Model (DEM). Retrieved from https://www.usgs.gov/the-national-map-data-delivery.
    - Highway Safety Department. (2023). Auto Accident data.
    - Utah Geospatial Resource Center (UGRC). (2023). Roads and Trails Dataset. Retrieved from https://gis.utah.gov/products/sgid/transportation/.
    - Mountain Project. (n.d.). Logan Canyon Climbing Locations. Retrieved from https://www.mountainproject.com/area/105739310/logan-canyon.
    - Dallin Moon. 2023. Backcountry Names Dataset.


## Future Improvements

 - The arcpy tool used to calculate the viewsheds is dated and there is another one available. This would increase the control over the units of the viewshed and also remove the need to use the buffer tool to clip the viewshed to the chosen radius. Instead the outer radius is an inherent part of the viewshed tool.
 - Increased control over inputs to eliminate the need to create a score field for each input parameter. Instead when selecting inputs for the scoring parameter the user will input a feature class and a number that will be the value for the score attribute. If a feature requires multiple score values it will still be necessary to prep the feature class before running the tool.

