###### If you find my repository helpful, please star⭐ it 🌟.
## Analyze and Visualize Looker Data: Challenge lab

 ### Task - 1 : Create Looks :-
##### Look - 1 : Most Heliports by State :-
Click Airports Explore.
- Click City and State Dimensions. Click Count Measure. Hover over the Facility Type, click the Filter By Field button (three lines), and set the Airports Facility type to is equal to HELIPORT.

In the data section, make the sure Airports Count column is in descending order. Clicking the column reorders it. For visualization, make sure the type is set to Table.

Save the visualization as a Look.

##### Look - 2 : Facility Type Breakdown :-

Click Airports Explore.
- Click State Dimensions. Click Count Measure. Hover over the Facility Type, click the Pivot button. Set the row limit to 10.

In the data section, make the sure Airports Facility Type column is in descending order. Clicking the column reorders it. For visualization, make sure the type is set to Table.

Save the visualization as a Look.
##### Look - 3 : Percentage Cancelled :-

Click Flights Explore.
- Click Aircraft Origin > City and Aircraft Origin > State Dimensions. - Under Flights Details, click Cancelled Count Measure.
- Under Flights, click the Count Measure. Next, hover over it and click the filter button.

Set the filter to: Flights Count is greater than 10000.

Click Run.

Click the + Add button next to Custom Fields on the left toolbar. Select Table Calculation. For your table calculation, add the formula in from above, rename it to Percentage of Flights Cancelled, change to Percent (3) formatting, and click save.

In the data section, make the sure Percent Cancelled column is in descending order. Clicking the column reorders it. Click the gear icon next to the Flights Count column and select "Hide from Visualization". Repeat this for the Cancelled Count Column. In the visualization pane, make sure you're using a table.

Save the visualization as a Look.
