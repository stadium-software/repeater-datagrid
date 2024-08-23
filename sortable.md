# Sortable
In order to allow users to click on column headers to make the DataGrid data sortable, we need to: 

1. Swap the Header *Label* controls into *Link* controls

![](images/HeaderLinkControls.png)

2. Create the *Click Event Handler* for each of the header *Link* controls
3. Drag the "GetData" script into the control *Click Event Handler* script

![](images/SortingEventHandler.png)