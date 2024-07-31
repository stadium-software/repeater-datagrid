# Repeater As DataGrid <!-- omit in toc -->
Using this module, you can use a *Repeater* control to create a server-side DataGrid that looks and works similar to the standard Stadium *DataGrid* control. 

The module comes with two scripts and two CSS files. The scripts provide functionality to facilitate the rendering, sorting and paging features. The CSS makes a Stadium *Repeater* control look like a DataGrid. The *Repeater* control must be custom built to look like the dataset that it will display. 

To illustrate this module, it comes with a sample application that displays data from a database table with 2 million records. This module can be configured to use any connector and data source, and works with data sources of any size. 

## Contents <!-- omit in toc -->
- [Version](#version)
- [Database Setup](#database-setup)
- [Setup](#setup)
  - [Application Setup](#application-setup)
  - [Connector Setup](#connector-setup)
  - [Queries Setup](#queries-setup)
  - [Types Setup](#types-setup)
  - [Page Setup](#page-setup)
    - [Container](#container)
    - [Grid](#grid)
    - [Repeater](#repeater)
  - [Global Script Setup](#global-script-setup)
    - [Initialising the module](#initialising-the-module)
    - [Querying the module](#querying-the-module)
  - [DataGrid Events](#datagrid-events)
    - [Sorting](#sorting)
    - [Paging](#paging)
    - [Link Columns](#link-columns)
  - [Page.Load Setup](#pageload-setup)
  - [CSS Setup](#css-setup)
- [Custom Filters](#custom-filters)
- [Showing Spinners](#showing-spinners)

# Version
1.0 initial

# Database Setup
The module can be configured to work with any data source and connector. 

The attached example application uses a database connector and queries. To run the sample application, you need to:
1. Create a database in a SQL Server instance called "StadiumLoadTest"
2. The unzip and run the SQL script in the database folder in this repo (this will create a table called "User") [script file](database/script.zip)

# Setup

## Application Setup
1. Check the *Enable Style Sheet* checkbox in the application properties

## Connector Setup
Set up your connector as you normally would. 

To run the example application, create a database connector to the database you [created above](#database-setup). 

## Queries Setup
The module requires two data sets: 

1. The total number of records
2. The data to be attached to the *Repeater*

Add the basic queries below to make the example application work complete with paging and sorting functions

Example "TotalRecords" Query
```sql
select count(ID) as total from [User]
```

Example "Select" Query
```sql
SELECT 
	ID
      ,name
      ,gender
      ,address
      ,birthdate
      ,adddatetime
  FROM [User]
  ORDER BY
  case when UPPER(@sortField) = 'ID' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN ID END ASC,
  case when UPPER(@sortField) = 'ID' AND LOWER(@sortDirection) = 'desc' THEN ID END DESC,
  case when LOWER(@sortField) = 'name' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN [name] END ASC,
  case when LOWER(@sortField) = 'name' AND LOWER(@sortDirection) = 'desc' THEN [name] END DESC,
  case when LOWER(@sortField) = 'gender' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN gender END ASC,
  case when LOWER(@sortField) = 'gender' AND LOWER(@sortDirection) = 'desc' THEN gender END DESC,
  case when LOWER(@sortField) = 'address' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN [address] END ASC,
  case when LOWER(@sortField) = 'address' AND LOWER(@sortDirection) = 'desc' THEN [address] END DESC,
  case when LOWER(@sortField) = 'birthdate' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN birthdate END ASC,
  case when LOWER(@sortField) = 'birthdate' AND LOWER(@sortDirection) = 'desc' THEN birthdate END DESC,
  case when @sortField = '' then ID end ASC,
  case when @sortField = 'undefined' then ID end ASC
OFFSET @offsetRows ROWS FETCH NEXT @pageSize ROWS ONLY
```

## Types Setup
Add a new type that contains all the columns in your dataset. 

The example dataset type is called "UserDG" and contains the following columns:
1. ID (Any)
2. name (Any)
3. gender (Any)
4. address (Any)
5. birthdate (Any)
6. adddatetime (Any)

![](images/ColumnType.png)

## Page Setup
To function correctly, the page must contain a number of controls. Some of these provide for DataGrid-specific functions, like paging, while others serve to simply display the data from your dataset. Each control set is defined in detail below. 

The final set of controls for the example application will look like this:

![](images/PageControls.png)

### Container
1. Drag a *Container* control to the page
2. Give it a suitable name (e.g. ServerSideDataGridContainer)
3. Add a class of your choice to the control *Classes* property to uniquely identify the control (e.g. server-side-datagrid)

### Grid
1. Drag a *Grid* control into the *Container* control
2. For each column you wish to display
   1. Drag a *Link* control into the *Grid* if the column must be sortable
   2. Drag a *Label* control into the *Grid* if the column should not be sortable
   3. Leave the column empty if you don't want to display a column header

![](images/GridHeaders.png)

### Repeater
1. Drag a *Repeater* control into the *Grid* control (under the header row)
2. Assign the *Type* you careated above to the *Repeater* *ListItem Type* property
![](images/RepeaterListItemType.png)
3. For each column you wish to display
   1. Drag a *Label* control into the *Grid*
   2. Map the correct ListItem Property to the *Label Text* property (example shows the "ID" Label)
![](images/BindingControlsToRepeater.png)

![](images/RepeaterColumns.png)

## Global Script Setup

### Initialising the module

### Querying the module

## DataGrid Events

### Sorting

### Paging

### Link Columns

## Page.Load Setup

## CSS Setup

# Custom Filters

# Showing Spinners

