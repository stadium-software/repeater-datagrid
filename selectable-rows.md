# Selectable Rows

## Setup

To enable users to select rows by checking a checkbox

1. Create an application *Variable* in the application properties called "SelectedRows"

![](images/CreateSessionVar.png)

2. Drag a *SetValue* action into the Page.Load event handler
   1. Target: Session.Variables.SelectedRows
   2. Value: Initialise the variable with an empty list
```javascript
= []
```

![](images/RowSelectInitEmptyList.png)

3. Add a column for a *CheckBox* control to the "ColumnsList" in the "Initialise" script with a sortable property of false

**For example**
```javascript
{
	"name": "SelectCheckbox",
	"header": "Selected",
	"sortable": false
}
```

![](images/ColumnsListAdd.png)

4. Drag a *CheckBox* control into the *Repeater* control

![](images/CheckBoxControl.png)

5. Create the control *Change* Event Handler

![](images/RowSelectChangeEvent.png)

6. In the *Change* Event Handler, add a *Decision*

![](images/RowSelectDecision.png)

7. Drag a *Java Script* action into the "If" branch of the *Decision*
8. Add the script below to the *Code* property of the action to add the value of the "IDLabel" to the list in the session variable

```javascript
Session.Variables.SelectedRows.push(parseInt(IDLabel.Text));
```

9. Drag a *Java Script* action into the "Else" branch of the *Decision*
10. Add the script below to the *Code* property of the action to remove the value of the "IDLabel" from the list in the session variable

```javascript
Session.Variables.SelectedRows.splice(Session.Variables.SelectedRows.indexOf(parseInt(IDLabel.Text)), 1);
```

## Checking Checkboxes On Load

To show some checkboxes as checked when the DataGrid is loaded

1. Add values to the 

![](images/SetSelectedSessionVar.png)

