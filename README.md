# Server-Side DataGrid Repeater <!-- omit in toc -->

## Contents <!-- omit in toc -->
- [Overview](#overview)
  - [Example Application](#example-application)
- [Version](#version)
- [Application Setup](#application-setup)
  - [Application Properties](#application-properties)
  - [Connector](#connector)
    - ["TotalRecords"](#totalrecords)
    - ["Select"](#select)
  - [Global Script](#global-script)
  - [Types](#types)
    - [Column](#column)
    - [State](#state)
    - [DataSet](#dataset)
  - [Page](#page)
    - [Container](#container)
    - [Grid](#grid)
    - [Repeater](#repeater)
    - [Labels](#labels)
  - [Page Scripts](#page-scripts)
    - ["GetData" Page Script](#getdata-page-script)
    - ["Initialise" Page Script](#initialise-page-script)
  - [Page.Load Event Handler](#pageload-event-handler)
  - [CSS](#css)
    - [Before v6.12](#before-v612)
    - [v6.12+](#v612)
    - [Customising CSS](#customising-css)
  - [Upgrading Stadium Repos](#upgrading-stadium-repos)
  - [Loading Spinners](#loading-spinners)
  - [Link Columns](#link-columns)
  - [Selectable Rows](#selectable-rows)
  - [Load Specific Page](#load-specific-page)
  - [Selectable Page Size](#selectable-page-size)
  - [Conditional Cell Styling](#conditional-cell-styling)
  - [Custom Filters](#custom-filters)
  - [Editable DataGrids](#editable-datagrids)
  - [Data Exports](#data-exports)

# Overview
The Stadium *DataGrid* loads all records a query or API return into memory as a JSON array of objects. When datasets are large, they take a long time to transmit across networks and client machines can run out of memory, causing browsers to freeze or even crash. One solution is to break large datasets into smaller units, here called "pages" of data. This cannot be achieved using the DataGrid control. 

An alternative to the DataGrid for the display of datasets is the Repeater control. This module provides a method for displaying data from sets in individual pages in the Repeater control that is made to look and function similar to the standard Stadium *DataGrid* control. Use this module when you want to give users access to datasets that are too large for the standard DataGrids. 

When using this module, the queries or WebServices limit the number of records they return to specific slices. This is done using parameters from the DataGrid in conjunction  with the paging techniques the respective databases support. The example below shows how this works with MSSQL databases. 

https://github.com/user-attachments/assets/c83d203c-d2de-45a4-bb21-5cca30a8f350

# Basic Setup Overview

1. Create your [Connectors](#connector) and DataSources (Queries or WebServices)
2. Add the [RepeaterDataGrid](#global-script) script
2. Add the [types](#types)
3. Add the [controls](#page) to your Page
4. Create the [Page Scripts](#page-scripts)
5. Create the [Page.Load](#pageload-event-handler) event handler
6. Add the [CSS](#css) to your Embedded files and reference them in the application  header property

Check out these advanced features

1. [Loading Spinners](#loading-spinners)
2. [Selectable Rows](#selectable-rows)
3. [Load Specific Page](#load-specific-page)
4. [Selectable Page Size](#selectable-page-size)
5. [Conditional Cell Styling](#conditional-cell-styling)
5. [Custom Filters](#custom-filters)
5. [Editable DataGrids](#editable-datagrids)
5. [Data Exports](#data-exports)

## Example Application
The repo includes the sample application shown in the video. 
[ServerSideRepeaterDataGrid.sapz](Stadium6/ServerSideRepeaterDataGrid.sapz?raw=true)

To run the example application, follow these steps:
1. Setup the Database
   1. [Follow these instructions](database-setup.md) to set up the database for the [sample application](Stadium6/ServerSideRepeaterDataGrid.sapz)
   2. Use the [data scripts](data/data.zip) to populate the *MyData* table with as many records as you wish
2. Open the [sample application](Stadium6/ServerSideRepeaterDataGrid.sapz)
   1. Amend the database connector
   2. Hit the *Preview* button

# Version
1.0 initial

1.1 Changed global script "Columns" input parameter name to "Headers", "ColumnsList" to "HeadersList" and the "Column" type name to "Header"

1.1.1 (CSS only) Added a few variables to the CSS for better customisation options

1.2 Added optional 'Classic Paging' option; changed "Headers" back to "Columns", "HeadersList" to "ColumnsList" and the "Header" type name to "Column"

1.3 Upgraded readme to 6.12+; converted px to rem; fixed sorting indicator bug; adjusted various display properties

1.3.1 Slight update CSS update (paging buttons backgrounds)

# Application Setup

## Application Properties
Check the *Enable Style Sheet* checkbox in the application properties

## Connector
To run the sample, a database connector to the "StadiumFilterData" database and two Queries need to be added

### "TotalRecords"
Create a query called "TotalRecords". The result is used when calculating the total number of pages in the *Repeater*
```sql
select count(ID) as total from MyData
```

### "Select"
Create a query called "Select"

**NOTE: When pasting this SQL into Stadium and pressing the "Fetch Fields & Parameters" button, an error will pop up. Set the "Type" option for the parameters called "offsetRows" and "pageSize" to "Int64" as shown below and press the "Fetch Fields & Parameters" button again.**

```sql
SELECT ID
      ,FirstName
      ,LastName
      ,NoOfChildren
      ,NoOfPets
      ,StartDate
      ,EndDate
      ,Healthy
      ,Happy
      ,Subscription
  FROM dbo.MyData
  ORDER BY
  case when UPPER(@sortField) = 'ID' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN ID END ASC,
  case when UPPER(@sortField) = 'ID' AND LOWER(@sortDirection) = 'desc' THEN ID END DESC,
  case when LOWER(@sortField) = 'FirstName' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN FirstName END ASC,
  case when LOWER(@sortField) = 'FirstName' AND LOWER(@sortDirection) = 'desc' THEN FirstName END DESC,
  case when LOWER(@sortField) = 'LastName' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN LastName END ASC,
  case when LOWER(@sortField) = 'LastName' AND LOWER(@sortDirection) = 'desc' THEN LastName END DESC,
  case when LOWER(@sortField) = 'NoOfChildren' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN NoOfChildren END ASC,
  case when LOWER(@sortField) = 'NoOfChildren' AND LOWER(@sortDirection) = 'desc' THEN NoOfChildren END DESC,
  case when LOWER(@sortField) = 'NoOfPets' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN NoOfPets END ASC,
  case when LOWER(@sortField) = 'NoOfPets' AND LOWER(@sortDirection) = 'desc' THEN NoOfPets END DESC,
  case when LOWER(@sortField) = 'StartDate' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN StartDate END ASC,
  case when LOWER(@sortField) = 'StartDate' AND LOWER(@sortDirection) = 'desc' THEN StartDate END DESC,
  case when LOWER(@sortField) = 'EndDate' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN EndDate END ASC,
  case when LOWER(@sortField) = 'EndDate' AND LOWER(@sortDirection) = 'desc' THEN EndDate END DESC,
  case when LOWER(@sortField) = 'Healthy' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN Healthy END ASC,
  case when LOWER(@sortField) = 'Healthy' AND LOWER(@sortDirection) = 'desc' THEN Healthy END DESC,
  case when LOWER(@sortField) = 'Happy' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN Happy END ASC,
  case when LOWER(@sortField) = 'Happy' AND LOWER(@sortDirection) = 'desc' THEN Happy END DESC,
  case when LOWER(@sortField) = 'Subscription' AND (LOWER(@sortDirection) = 'asc' OR @sortDirection = '') THEN Subscription END ASC,
  case when LOWER(@sortField) = 'Subscription' AND LOWER(@sortDirection) = 'desc' THEN Subscription END DESC,
  case when @sortField = '' then ID end ASC,
  case when @sortField = 'undefined' then ID end ASC
  OFFSET @offsetRows ROWS FETCH NEXT @pageSize ROWS ONLY
```

## Global Script
1. Create a Global Script called "RepeaterDataGrid"
2. Add the input parameters below to the Global Script
   1. Columns
   2. ContainerClass
   3. EditableGrid
   4. EventCallback
   5. State
   6. TotalRecords
   7. PagingType
3. Drag a *JavaScript* action into the script
4. Add the Javascript below unchanged into the JavaScript code property
```javascript
/* Stadium Script v1.3 Init https://github.com/stadium-software/repeater-datagrid */
let scope = this;
let cols = ~.Parameters.Input.Columns;
let eventHandler = ~.Parameters.Input.EventCallback;
let state = ~.Parameters.Input.State;
let pageSize = parseInt(state.pageSize);
let sortField = state.sortField;
let sortDirection = state.sortDirection;
let pagingType = ~.Parameters.Input.PagingType || "default";
let pagers = 10;
let page = parseInt(state.page);
let totalRecords = parseInt(~.Parameters.Input.TotalRecords);
if (isNaN(page)) page = 1;
let totalPages = Math.ceil(totalRecords / pageSize);
let containerClass = ~.Parameters.Input.ContainerClass;
if (!containerClass) {
     console.error("The ContainerClass parameter is required");
     return false;
}
let container = document.querySelectorAll("." + containerClass);
if (container.length == 0) {
    console.error("The class '" + containerClass + "' is not assigned to any container");
    return false;
} else if (container.length > 1) {
    console.error("The class '" + containerClass + "' is assigned to multiple containers");
    return false;
}
container = container[0];
container.classList.add("stadium-dg-repeater");
let grid = container.querySelectorAll(".grid-layout");
if (grid.length == 0) {
    console.error("The container '" + containerClass + "' must contain a Grid control");
    return false;
} else if (grid.length > 1) {
    console.error("The container '" + containerClass + "' must contain only one Grid control");
    return false;
}
grid = grid[0];
let contID = container.id;
let cellsPerRow = cols.length;
if (document.getElementById(contID + "_stylesheet")) document.getElementById(contID + "_stylesheet").remove();
attachStyling();
addHeaders(cols);
addPaging();
let cells = container.querySelectorAll(".grid-repeater-item");
let rowNo = 1, cellCounter = 0;
for (let i = 0; i < cells.length; i++) {
    cellCounter++;
    cells[i].setAttribute("row-no", rowNo);
    if (cellCounter >= cellsPerRow) {
        cellCounter = 0;
        rowNo++;
    }
}
/*----------------------------------------------------------------------------------------------*/
function addHeaders(c) { 
    let headers = container.querySelectorAll(".repeater-header");
    for (let i = 0; i < headers.length; i++) {
        headers[i].remove();
    }
    for (let i = c.length-1; i > -1; i--) {
        let gItem = createTag("div", ["grid-item", "repeater-header"], []);
        let el = createTag("div"), inner;
        if (c[i].header && c[i].hasOwnProperty('sortable') && c[i].sortable !== false && c[i].sortable !== "false") {
            el = createTag("div", ["control-container", "link-container"]);
            inner = createTag("a", ["btn", "btn-lg", "btn-link"], [{name: "rel", value:"noopener noreferrer"}, {name: "field", value:c[i].name}], c[i].header);
            inner.addEventListener("click", handleSort);
            el.appendChild(inner);
        } else if (c[i].header) { 
            el = createTag("div", ["control-container", "label-container"]);
            inner = createTag("span", [], [], c[i].header);
            el.appendChild(inner);
        }
        if (c[i].hasOwnProperty('visible') && c[i].visible == false || c[i].visible == 'false') el.classList.add("visually-hidden");
        if (c[i].name == sortField && sortDirection.toLowerCase() == "asc") el.classList.add("dg-asc-sorting");
        if (c[i].name == sortField && sortDirection.toLowerCase() == "desc") el.classList.add("dg-desc-sorting");
        gItem.appendChild(el);
        grid.insertBefore(gItem, grid.firstChild);
    }
}
function addPaging() { 
    if (container.querySelector(".paging")) container.querySelector(".paging").remove();
    let pagingContainer = createTag("div", ["layout-control", "container-layout", "paging", "inline-block-element"], []);
    if (pagingType.toLowerCase() == "classic") {
        let pagingButtonsContainer = createTag("ul", ["pagination"], []);
        let pagingButtons = addClassicPagingButtons(pagingButtonsContainer, page);
        pagingContainer.appendChild(pagingButtons);
    } else {
        let prevButtonContainer = createTag("div", ["control-container", "button-container", "previous-button"], []);
        let prevButton = createTag("div", ["btn", "btn-lg", "btn-default"], [], "<<");
        let nextButtonContainer = createTag("div", ["control-container", "button-container", "next-button"], []);
        let nextButton = createTag("div", ["btn", "btn-lg", "btn-default"], [], ">>");
        let goInputContainer = createTag("div", ["control-container", "text-box-container", "specific-page"], []);
        let goInput = createTag("input", ["form-control", "error-border", "text-box-input", "specific-page-input"], [{name: "type", value: "text"}]);
        let goButtonContainer = createTag("div", ["control-container", "button-container", "specific-page-go"], []);
        let goButton = createTag("div", ["btn", "btn-lg", "btn-default"], [], "Go");
        let pageInfoContainer = createTag("div", ["control-container", "label-container", "page-info"]);
        let pageInfo = createTag("span", [], [], getPageLabel());
        if (page == 1) prevButtonContainer.classList.add("disabled");
        if (page == totalPages) nextButtonContainer.classList.add("disabled");
        prevButtonContainer.appendChild(prevButton);
        nextButtonContainer.appendChild(nextButton);
        goInputContainer.appendChild(goInput);
        goButtonContainer.appendChild(goButton);
        pageInfoContainer.appendChild(pageInfo);

        pagingContainer.appendChild(prevButtonContainer);
        pagingContainer.appendChild(nextButtonContainer);
        pagingContainer.appendChild(goInputContainer);
        pagingContainer.appendChild(goButtonContainer);
        pagingContainer.appendChild(pageInfoContainer);

        prevButton.addEventListener("click", function(){
            handlePaging("previous");
        });
        nextButton.addEventListener("click", function(){
            handlePaging("next");
        });
        goButton.addEventListener("click", function(){
            handlePaging("go");
        });
    }

    let stackLayout, allStacks = container.querySelectorAll(".stack-layout-container");
    for (let i = 0; i < allStacks.length; i++) {
        if (allStacks[i].contains(grid) && allStacks.length > i) {
            stackLayout = allStacks[i + 1];
        }
    }
    if (!stackLayout) {
        stackLayout = createTag("div", ["stack-layout-container"]);
        container.appendChild(stackLayout);
    }
    stackLayout.classList.add('paging-stack-layout');
    stackLayout.insertBefore(pagingContainer, stackLayout.firstChild);
}
function handlePaging(tp) { 
    let nextBtn = container.querySelector(".next-button"),
        prevBtn = container.querySelector(".previous-button"),
        goInpt = container.querySelector(".specific-page-input"),
        pageInfo = container.querySelector(".page-info span");
    nextBtn.classList.remove("disabled");
    prevBtn.classList.remove("disabled");
    if (tp == "next" && page < totalPages) page++;
    if (tp == "previous" && page > 1) page--;
    if (tp == "go") {
        let pgVal = goInpt.value;
        goInpt.value = "";
        if (!isNaN(pgVal) && pgVal >= 1 && pgVal <= totalPages) {
            page = parseInt(pgVal);
        }
    }
    if (page == 1) prevBtn.classList.add("disabled");
    if (page == totalPages) nextBtn.classList.add("disabled");
    pageInfo.textContent = getPageLabel();
    scope[eventHandler]({ sortField: sortField, sortDirection: sortDirection, page: page, pageSize: pageSize });
}
function addClassicPagingButtons(parent, currentPage) {
    currentPage = currentPage - 1;
    let firstPage = currentPage - (currentPage % 10);
    let lastPage = firstPage + pagers;
    parent.innerHTML = "";
    if (totalPages < lastPage) lastPage = totalPages;
    if (firstPage >= pagers) {
        let pagingButton = createTag("li", [], []);
        let pagingButtonInner = createTag("a", [], [], "«");
        pagingButtonInner.addEventListener("click", handleClassicPaging);
        pagingButton.appendChild(pagingButtonInner);
        parent.appendChild(pagingButton);
    }
    for (let i = firstPage; i < lastPage; i++) {
        let pagingButton = createTag("li", [], []);
        if (i == currentPage) pagingButton.classList.add('active');
        let pagingButtonInner = createTag("a", [], [], i + 1);
        pagingButtonInner.addEventListener("click", handleClassicPaging);
        pagingButton.appendChild(pagingButtonInner);
        parent.appendChild(pagingButton);
    }
    if (totalPages > (firstPage + pagers)) {
        let pagingButton = createTag("li", [], []);
        let pagingButtonInner = createTag("a", [], [], "»");
        pagingButtonInner.addEventListener("click", handleClassicPaging);
        pagingButton.appendChild(pagingButtonInner);
        parent.appendChild(pagingButton);
    }
    return parent;
}
function handleClassicPaging(ev) {
    let el = ev.target;
    let elParent = el.closest("li");
    if (el.classList.contains("active")) {
        return false;
    }
    let paging = container.querySelector(".paging");
    let pagination = container.querySelector(".pagination");
    paging.querySelector(".active").classList.remove("active");
    let pg = el.textContent;
    if (pg == "»") {
        page = page % 10 === 0 ? page + 1 : page + pagers - (page % 10) + 1;
        if (page > totalPages) page = totalPages - (totalPages % 10) + 1;
        let pagingButtons = addClassicPagingButtons(pagination, page);
        paging.appendChild(pagingButtons);
    } else if (pg == "«") {
        page = page % 10 === 0 ? page - pagers : page - (page % 10);
        let pagingButtons = addClassicPagingButtons(pagination, page);
        paging.appendChild(pagingButtons);
    }
    else {
        page = Number(pg);
        elParent.classList.add("active");
    }
    scope[eventHandler]({ sortField: sortField, sortDirection: sortDirection, page: page, pageSize: pageSize });
}
function handleSort(e) { 
    let clickedEl = e.target;
    let colHead = clickedEl.getAttribute("field");
    let dir = "asc";
    if (clickedEl.closest(".dg-asc-sorting")) {
        dir = "desc";
    }
    sort(colHead, dir);
}
function sort(field, direction) {
    if (!["asc", "desc"].includes(direction.toLowerCase())) direction = "asc";
    let allHeaders = container.querySelectorAll(".repeater-header .link-container");
    for (let i = 0; i < allHeaders.length; i++) {
        allHeaders[i].classList.remove("dg-asc-sorting", "dg-desc-sorting");
    }
    grid.querySelector(".repeater-header .link-container:has(.btn-link[field='" + field + "'])").classList.add("dg-" + direction + "-sorting");
    sortDirection = direction;
    sortField = field;
    scope[eventHandler]({ sortField: field, sortDirection: direction, page: page, pageSize: pageSize });
}
function createTag(type, arrClasses, arrAttributes, text) {
    let el = document.createElement(type);
    if (arrClasses && arrClasses.length > 0) {
        arrClasses = arrClasses.filter(function( element ) { return element !== undefined; });
        let cl = el.classList;
        cl.add.apply(cl, arrClasses);
    }
    if (arrAttributes && arrAttributes.length > 0) { 
        for (let i = 0; i < arrAttributes.length; i++) { 
            if (arrAttributes[i].name) el.setAttribute(arrAttributes[i].name, arrAttributes[i].value);
        }
    }
    if (text) el.textContent = text;
    return el;
}
function getPageLabel() {
    let pgLabel = "No records found";
    if (totalPages > 0) {
        pgLabel = "Page " + page.toLocaleString() + " of " + totalPages.toLocaleString();
    }
    return pgLabel;
}
function attachStyling() {
    let selector = [];
    for (let i = 0; i < cellsPerRow; i++) {
        selector.push(".grid-repeater-item:nth-child(" + (cellsPerRow * 2) + "n+" + (i + 1) + ")");
    }
    let css = `
#${contID} {
    .grid-item:nth-child(${cellsPerRow}n+1) {
        border-left: 1px solid var(--dg-border-color, var(--DATA-GRID-BORDER-COLOR, #ccc));
    }
    .grid-item:nth-child(${cellsPerRow}n) {
        border-right: 1px solid var(--dg-border-color, var(--DATA-GRID-BORDER-COLOR, #ccc));
    }
    ${selector.join(", ")} {
        background-color: var(--dg-alternate-row-bg-color, var(--DATA-GRID-ODD-ROW-BACKGROUND-COLOR));
    }
}`;
    let head = document.head || document.getElementsByTagName('head')[0], style = document.createElement('style');
    head.appendChild(style);
    style.type = 'text/css';
    style.id = contID + "_stylesheet";
    style.appendChild(document.createTextNode(css));
}
```

## Types
Create three types
1. Column
2. State
3. DataSet

### Column
This type is used to define the columns in the DataGrid
1. name (any)
2. header (any)
3. visible (any)
4. sortable (any)

![](images/ColumnType.png)

### State
The state of the DataGrid is stored in this type
1. pageSize (any)
2. page (any)
3. sortDirection (any)
4. sortField (any)

![](images/Statetype.png)

### DataSet
This type will be used in the *Repeater* *ListItem Type* property. The "DataSet" type must contain properties for all columns the DataGrid you create (visible and hidden). 

The "DataSet" type for the sample application as the following properties
1. ID (any)
2. FirstName (any)
3. LastName (any)
4. NoOfChildren (any)
5. NoOfPets (any)
6. StartDate (any)
7. EndDate (any)
8. Healthy (any)
9. Happy (any)
10. Subscription (any)

![](images/DataSetType.png)

## Page
The page must contain a number of controls

![](images/AllControls.png)

### Container
A *Container* is the wrapper for all DataGrid controls
1. Drag a *Container* control to the page and give it a suitable name (e.g. DataGridContainer)
3. Add a class of your choice to the control *Classes* property to uniquely identify the control in the application (e.g. server-side-datagrid)

### Grid
A *Grid* control will create the DataGrid rows and columns
1. Drag a *Grid* control into the *Container* control

### Repeater
A *Repeater* control will contain the data (rows) in the DataGrid
1. Drag a *Repeater* control into the *Grid* control
2. Assign the "DataSet" *Type* to the *Repeater* *ListItem Type* property

![](images/RepeaterListItemType.png)

### Labels
*Label* controls in the *Repeater* represent the columns in the DataGrid
1. For each column (field) in your DataSet
   1. Drag a *Label* control into the *Repeater*
   2. Name the *Label* "*ColumnName*Label"
   3. In the *Label* *Text* property, select the corresponding *Repeater ListItem* property in the dropdown (see screenshot below)
   4. Set the *Visible* property of the *Label* to "false" to hide the column

![](images/BindingControlsToRepeater.png)

## Page Scripts

### "GetData" Page Script
Create a script under the page called "GetData" with the input Parameter:
1. State

**Script Actions**

![](images/GetDataScript.png)

1. Drag a *Variable* to the script and call it "OffsetRows_var"
2. In the "OffsetRows_var" *Value* property, paste the following value (including the =) to calculate the offset
```javascript
= (~.Parameters.Input.State.page - 1) * ~.Parameters.Input.State.pageSize
```
3. Drag the "StadiumFilterData_Select" query to the script and paste the values below into the query input parameters
   1. sortField: 
   ```javascript
   = ~.Parameters.Input.State.sortField
   ```
   2. sortDirection: 
   ```javascript
   = ~.Parameters.Input.State.sortDirection
   ```
   3. offsetRows: 
   ```javascript
   = ~.OffsetRows_var
   ```
   4. pageSize: 
   ```javascript
   = ~.Parameters.Input.State.pageSize
   ```

4. Drag a *SetValue* to the script to set the *Repeater* data
   1. Target: The Repeater List Property
   2. Source: The data returned by the connector

![](images/SetRepeaterData.png)

### "Initialise" Page Script
Create a script under the page called "Initialise" with the input Parameter:
1. State

**Script Actions**

![](images/InitialiseScript.png)

1. Drag a "State" *Type* into the script
   1. In the *Value* property of the "State" *Type*, assign the script input parameter called "State"

![](images/StateInputAssignment.png)

2. Drag the "TotalRecords" query into the "Initialise" script
3. Drag the "GetData" script into the "Initialise" script
   1. Assign the "State" type to the "State" input parameter of the "GetData" script

![](images/GetDataInputAssign.png)

4. Drag a *List* into the script and call it "ColumnsList"
5. Assign the "Column" *Type* to the *Item Type* property
6. Add an entry in this list for **every column** in your *Repeater* DataGrid
   1. name (required): The column name (case sensitive)
   2. header (optional): The header title shown on this column. A value is necessary for users to be able to sort by the column
   3. visible (optional): Add "false" to hide the column (default is true)
   4. sortable (optional): Add "false" to show the heading as an (unclickable) *Label* instead of a *Link* (default is true)

**Example ColumnsList Value**
```json
[{
 "name": "ID",
 "header": "ID"
},{
 "name": "FirstName",
 "header": "First Name",
 "visible": false,
 "sortable": false
},{
 "name": "LastName",
 "header": "Last Name"
},{
 "name": "NoOfChildren",
 "header": "Children"
},{
 "name": "NoOfPets",
 "header": "Pets"
},{
 "name": "StartDate",
 "header": "Start Date"
},{
 "name": "EndDate",
 "header": "End Date"
},{
 "name": "Healthy",
 "header": "Healthy"
},{
 "name": "Happy",
 "header": "Happy"
},{
 "name": "Subscription",
 "header": "Subscription"
}]
```

5. Drag the "RepeaterDataGrid" script into the "Initialise" script and provide the "RepeaterDataGrid" input parameters
   1. Columns: The *List* of columns called "ColumnsList"
   2. ContainerClass: The unique class you assigned to the main container (e.g. server-side-datagrid)
   3. EventCallback: The name of the script that fetches and assigns the data pages to the *Repeater*. In the example application this is called "GetData"
   4. State: The "State" *Type* created in step 1 of the "Initialise" script
   5. EditableGrid (optional): Ignore this property for standard data display. It's a boolean that hides the paging controls and changes header *Links* controls into *Label* controls ([see Editable Datagrids](#editable-datagrids))
   6. TotalRecords: The "total" result returned by the "TotalRecords" query: 
```javascript
~.StadiumFilterData_Totals.FirstResult.total
```
   7. PagingType (optional): Leave blank or enter 'classic' for the standard Stadium DataGrid paging format

![](images/RepeaterDGScriptInputParams.png)

## Page.Load Event Handler

![](images/PageLoadEv.png)

1. Drag a "State" *Type* into the event handler
2. Open the *Object Editor* in the dropdown of the "State" *Values* property
3. Assign values to the properties
   1. pageSize: The number of items you want to show in each DataGrid page
   2. page: The initial page to show in the DataGrid
   3. sortDirection: The initial sort direction (asc or desc)
   4. sortField: The initial sort field of the dataset

![](images/StateTypeObjectProps.png)

4. Drag the "Initialise" script into the Page.Load event handler
   1. Assign the "State" type to the "State" input parameter

## CSS
The CSS below is required for the correct functioning of the module. Variables exposed in the [*css-file-variables.css*](css-file-variables.css) file can be [customised](#customising-css).

### Before v6.12
1. Create a folder called "CSS" inside of your Embedded Files in your application
2. Drag the two CSS files from this repo [*css-file-variables.css*](css-file-variables.css) and [*css-file.css*](css-file.css) into that folder
3. Paste the link tags below into the *head* property of your application
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/css-file.css">
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/css-file-variables.css">
``` 

### v6.12+
1. Create a folder called "CSS" inside of your Embedded Files in your application
2. Drag the CSS files from this repo [*css-file.css*](css-file.css) into that folder
3. Paste the link tag below into the *head* property of your application
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/css-file.css">
``` 

### Customising CSS
1. Open the CSS file called [*css-file-variables.css*](css-file-variables.css) from this repo
2. Adjust the variables in the *:root* element as you see fit
3. Stadium 6.12+ users can comment out any variable they do **not** want to customise
4. Add the [*css-file-variables.css*](css-file-variables.css) to the "CSS" folder in the EmbeddedFiles (overwrite)
5. Paste the link tag below into the *head* property of your application (if you don't already have it there)
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/css-file-variables.css">
``` 
6. Add the file to the "CSS" inside of your Embedded Files in your application

**NOTE: Do not change any of the CSS in the 'css-file.css' file**

## Upgrading Stadium Repos
Stadium Repos are not static. They change as additional features are added and bugs are fixed. Using the right method to work with Stadium Repos allows for upgrading them in a controlled manner. 

How to use and update application repos is described here: [Working with Stadium Repos](https://github.com/stadium-software/samples-upgrading)# Optional Extensions

## Loading Spinners
To add a loading spinner to the DataGrid implement the [Spinners Module](https://github.com/stadium-software/spinners)

## Link Columns
[Link Columns](link-columns.md)

![](images/LinkColumnView.gif)

## Selectable Rows
[Selectable Rows](selectable-rows.md)

![](images/SelectableRowsVw.gif)

## Load Specific Page
[Loading a specific page](load-specific-page.md)

![](images/CustomLoadView.gif)

## Selectable Page Size
[Selectable Page Size](customisable-page-size.md)

![](images/SettablePgSize.gif)

## Conditional Cell Styling
[Conditional Cell Styling](conditional-cell-styling.md)

![](images/ConsitionalCellFormatView.gif)

## Custom Filters
[Custom Filters](custom-filters.md)

![](images/CutomFilterView.gif)

## Editable DataGrids
[Making columns or the entire DataGrid editable](editable.md)

![](images/EditingViewBoth.gif)

## Data Exports
[Data Exports](export.md)