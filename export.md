# Export
Enabling data export to CSV can be achieved with the use of the [List to CSV Download](https://github.com/stadium-software/utils-list-to-csv-download) repo

## Export Page Script

**Caution: Retrieving and exporting large data sets from a database or API can cause the browser to hang or crash**

1. Fetch the data to be exported by executing a query in an event handler or by calling a WebService
2. Assign the results set to the List input property of the *List to CSV Download* module script

## Multiple Files Export

**CAUTION: Pushing many files to the user using this method can also cause the browser to hang or crash.**

1. Use a "While" loop that fetches the data in smaller batches by assigning a export file to the pageSize query parameter
2. Push each batch to the "List to CSV Download" script. This will cause users to receive a number of files, rather than just one
3. In Chrome users will be asked to verify the download of multiple files