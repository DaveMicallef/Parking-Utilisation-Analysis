# Parking-Utilisation-Analysis
A tool for processing exported shapefiles and tables from GIS to analyse parking occupancy and utilisation. 

This repository provides tools for processing and analysing parking occupancy and utilisation data using shapefiles. The goal is to visualise parking bays, record occupancy occurrences, and generate insights into parking capacity and utilisation over time.

## Features
- Plotting: Visualise parking bays and occurrences accurately on a map.
- Capacity Calculations: Automatically calculate the capacity of parking bays based on area and type.
- Utilisation Analysis: Generate reports on occupancy and utilization for informed decision-making.

## Process
1. Plot the Parking Bays
Import and plot parking bay polygons as accurately as possible. Include an attribute column type to differentiate bays by marking and restrictions:
  - White: Public bays.
  - Yellow: Restricted bays (e.g., loading/unloading, residential garage, etc.).
  - Green: EV charging.
  - Blue: For disabled cardholders.
Add another attribute to indicate whether the bay is a single-car bay or an undivided bay with capacity for multiple cars but no physical dividers.

2. Plot Parking Occurrences:
- Plot points for each parked car.
- If multiple survey rounds are conducted, create separate layers for each (e.g., one layer for 11:00 AM, another for 12:00 PM).
- Ensure each occurrence is mapped within the correct geographic location.

3. Define Study Boundaries (Optional)
If applicable, add a polygon layer to define boundaries for specific analysis zones (e.g., by street, neighbourhood, or custom areas).

4. Calculate bay capacity
- i. Create a calculated column to calculate bay's area: !Shape!.getArea("GEODESIC","SQUAREMETERS")
- ii. Create a new calculated column and use the function below to determine the capacity of each bay based on type and area. This function considers local regulations to estimate capacity for undivided bays. Note that it makes use of the different bay classifications (e.g. white-single, white-undivided-parallel, etc.)

```python
def calc_capacity(parking_type, geo_area, description):
      # Define constants for dimensions (in meters)
      PARALLEL_WIDTH = 2
      PARALLEL_END_LENGTH = 4.8
      PARALLEL_BETWEEN_LENGTH = 6
      
      PERP_ANG_WIDTH = 2.4
      PERP_LENGTH = 4.8
      ANGLED_LENGTH = 6
      
      # Capacity calculation logic
      if parking_type in ['white-single', 'yellow-single'] or (parking_type == 'white-single' and description == 'mc'):
          capacity = 1
      
      elif parking_type in ['white-undivided-parallel', 'yellow-undivided']:   
          capacity = math.ceil(
              ((geo_area - 2 * (PARALLEL_END_LENGTH * PARALLEL_WIDTH)) /
              (PARALLEL_BETWEEN_LENGTH * PARALLEL_WIDTH)) + 2)
      
      elif parking_type == 'white-undivided-perpendicular':
          capacity = math.ceil(geo_area / (PERP_LENGTH * PERP_ANG_WIDTH))
      
      elif parking_type == 'white-undivided-angled':
          capacity = math.ceil(geo_area / (ANGLED_LENGTH * PERP_ANG_WIDTH))
      
      elif parking_type in ['bluebadge', 'bluebadge-reserved']:
          capacity = 1
      
      else:
          capacity = 0
      
      return capacity

  ```
5. Spatial Join the Parking Bays layer to the Zones/Streets layer
6. Batch Spatial Join with Parking Bays layer as target features and the multiple occupancy layers as Join features. Use _Match Option = Completely Contains_
7. Batch export the layers as .csv files and run the analysis notebook making the necessary amendments

### Illegal parking
To include occurances of illegal parking, follow the steps: 
1. Run the Spatial Join Tool:
- Go to Analysis > Tools > Spatial Join.
- Select your points layer as the Target Feature and your bays layer as the Join Feature.
- Choose the Join Operation as One to One or One to Many based on your needs.
- Set the Match Option to INTERSECT.

This will create a new layer where each point has information about the bays it intersects with.

2. Filter Out Points Outside of Bays:
- In the resulting Spatial Join layer, points that do not intersect with any bay will have null values in the attributes related to the bays.
- Open the attribute table of the Spatial Join result.
- Use the Select by Attributes tool to select all points where the bay-related attribute is null.
- These selected points are those that fall outside any designated parking bay.

3. Export the Selected Points as a table to be included in the analysis.

Contact me on davidwmicallef@gmail.com for any assistance! 
