# Export
The purpose of this DataGrid is to allow for the display of datasets that are too large to be shown in the standard DataGrid. So, the export of data to CSV is not a built-in feature of this DataGrid because datasets can be assumed to also be too large to be pushed to a CSV file for download. 

However, an option to enable the download of datasets as CSV files can be added by using the [List to CSV Download](https://github.com/stadium-software/utils-list-to-csv-download) repo. Note that retrieving and exporting large data sets from a database or API can cause the browser to hang or crash. 

## Export Page Script

1. Add a button or image control to a page
2. Add a Click event handler to the control
3. In the event handler
   1. Execute a connector query or API call to fetch the data to be exported
   2. Assign the results set to the List input property of the *List to CSV Download* module script

**Caution: Retrieving and exporting large data sets from a database or API can cause the browser to hang or crash**

![](images/SimpleExport.png)

## Multiple Files Export
Larger sets of data can be split into batches. This will result in the download of multiple export files by the user. 

1. Get the total number of records in the dataset by dragging the "TotalRecords" query to the script
2. Drag a variable to the script, call it "Counter_var" and set the initial value to 0
3. Drag a variable to the script, call it "PageSize_var" and set the initial value to number of records you wish to export in each file (e.g. 1000)
4. Drag a "While" loop to the script with the condition: 
```javascript
~.Counter_var <= ~.StadiumFilterData_Totals.FirstResult.total
```
5. In the While loop
   1. Drag the relevant "Select" query into the "While" loop and assign the "PageSize_var" to the "pageSize" query parameter
   2. Push each batch to the "List to CSV Download" script inside the "While" loop
   3. Drag a SetValue into the loop
      1. Target: ~.Counter_var
      2. Source: ~.Counter_var + ~.PageSize_var
   4. In Chrome users will be asked to verify the download of multiple files

**CAUTION: Pushing many files to the user using this method can cause the browser to hang or crash**

![](images/ExportScriptActions.png)