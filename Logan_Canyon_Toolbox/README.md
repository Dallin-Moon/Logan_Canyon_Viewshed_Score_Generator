# Logan Canyon Viewshed Score Calculator Toolbox — Instructions

## Overview

This tool calculates signal coverage from proposed cell tower locations and assigns each tower a score based on how many target features (e.g., critical areas) fall within its viewshed. A higher score represents better visibility.

While the current logic rewards high visibility, the system can be adapted to scenarios where **low visibility is preferable** (e.g., for conservation).

### Inputs Required

- **Observer Points**  
  - Must include a user-defined height in a field named `OFFSETA`.

- **Elevation Raster**

- **List of Scoring Parameters**  
  - Each must include a numeric field named `score` (capitalization doesn't matter).
  - If a feature class contains multiple score values, it will be handled differently (see [Data Preparation](#data-preparation)).

The tool performs the following steps:

1. Generates individual viewsheds from each observer point.
2. Converts viewsheds to vector polygons.
3. Uses a **Spatial Join** to intersect each polygon with the scoring parameters.
4. Updates the observer point attribute table with:
   - A final score
   - A breakdown of scoring contributions by parameter

> **Note:** The input Observer Points layer **will be modified** by the tool and is also part of the output.

---

## Data Preparation

1. **Projection**
   - Project all inputs to the **same coordinate system**.
   - The tool depends on consistent units for accurate analysis.

2. **Observer Points**
   - Must have a field named `OFFSETA` that defines the height of the point above the surface.
   - The tool calculates visibility using the DEM elevation + `OFFSETA`.

3. **Scoring Parameters**
   - Must contain a field called `score` (case-insensitive).
   - This field must be numeric (integer, float, etc.).
   - If features have **multiple score values**, the tool selects the **maximum** score among overlapping features.
     - Example: If three highway segments with scores 1, 3, and 5 intersect a viewshed, only 5 is added to the score.

4. **Output Location**
   - Define a geodatabase or folder for output data.
   - Input data will not be moved, but updated outputs (including polygons and observer points) will be saved here.

> It is recommended to use the same folder or geodatabase that stores your input observer points.

---

## Using the Tool

1. Launch the tool and input your prepared datasets.
2. Let it run (performance depends on system specs).

> **Performance Note:**  
> - On an Intel Core i7 with 16GB RAM and a 6GB GPU:  
>   - 10 points → ~5 minutes  
>   - 55 points → ~40 minutes

### Outputs

- A polygon layer of viewsheds.
- The updated Observer Points layer with new fields showing:
  - Final score
  - Individual contributions from each scoring parameter

> The `OBJECTID` of points will match the `OBJECTID` of their corresponding polygon in the viewshed output.

---

## Understanding the Output

You’ll receive a table like this:

| ID | F-Type | OFFSETA | Score | Parameter 1       | Parameter 2       | Parameter 3       | Parameter 4        | Parameter 5       |
|----|--------|---------|-------|-------------------|-------------------|-------------------|--------------------|-------------------|
| 1  | Point  | 20      | 15    | Score: 0 Count: 0 | Score: 3 Count: 1 | Score: 0 Count: 0 | Score: 10 Count: 2 | Score: 2 Count: 1 |
| 2  | Point  | 20      | 10    | Score: 0 Count: 0 | Score: 3 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 2 Count: 1 |
| 3  | Point  | 20      | 18    | Score: 0 Count: 0 | Score: 4 Count: 1 | Score: 0 Count: 0 | Score: 10 Count: 2 | Score: 4 Count: 2 |
| 4  | Point  | 20      | 16    | Score: 3 Count: 1 | Score: 4 Count: 1 | Score: 0 Count: 0 | Score: 5 Count: 1  | Score: 4 Count: 2 |

### Breakdown

- **OFFSETA**: Height used in viewshed calculation.
- **Score**: Final summed score from intersected features.
- **Each parameter field** follows this format:  
  - `Score: ## Count: ##`  
  - Score = value attributed by that parameter  
  - Count = number of intersected features contributing that score

> Multi-score parameters (e.g., accident-prone highway segments) will always show `Count: 1` and the **maximum** intersecting score.

---

## Limitations to Consider

This tool attempts to reduce uncertainty caused by:

- **Overrepresentation of polyline features**  
  - Polyline inputs are **dissolved** unless they contain multiple unique score values.
  - This avoids inflating scores when many small segments overlap a viewshed.

- **DEM generalizations**  
  - Viewshed outputs may have many small fragments.
  - These are aggregated to avoid double-counting and to clean the polygon geometry.

- **User-defined scores**  
  - It is the user’s responsibility to assign meaningful and appropriate score values to each feature class.

---

## Data Credits

- **USGS** (2022). Land Cover Data.  
  https://www.sciencebase.gov/catalog/item  
- **USGS** (2024). Digital Elevation Model (DEM).  
  https://www.usgs.gov/the-national-map-data-delivery  
- **Highway Safety Department** (2023). Auto Accident Data  
- **Utah Geospatial Resource Center (UGRC)** (2023). Roads and Trails Dataset  
  https://gis.utah.gov/products/sgid/transportation/  
- **Mountain Project**. Logan Canyon Climbing Locations  
  https://www.mountainproject.com/area/105739310/logan-canyon  
- **Dallin Moon** (2023). Backcountry Names Dataset

---

## Future Improvements and Expansions

- **Use cases beyond cell towers**  
  - Light pollution studies, billboard placement, viewshed protection, etc.
  - In these cases, **low visibility scores** could be ideal.

- **Use newer Viewshed2 tool**  
  - Provides better control over units and eliminates need for a buffer.

- **Enhanced input handling**  
  - Allow users to specify a static score value during input selection instead of requiring a score field.
  - Multi-score features would still need preprocessing.


