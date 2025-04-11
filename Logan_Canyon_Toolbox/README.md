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

## Limitations to Consider

## Data Credits

## Future Improvements

