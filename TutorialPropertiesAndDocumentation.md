# Tutorial #

![http://flexigrid.googlecode.com/files/flexigrid.jpg](http://flexigrid.googlecode.com/files/flexigrid.jpg)

As you can see the component is very intuitive and includes features to:
  * Search for records matching your supplied criteria (by first clicking the search icon).
  * Sort in either ascending or descending order by a selected column.
  * Hide and show columns to make optimum use of available space.
  * Navigate between pages using the navigation icons or jump straight to a particular page.

## How to Use ##

Adding the Flexigrid to your webpage couldn't be easier. Just download the code from http://www.flexigrid.info and copy the required files into your site's directories. You must also have a version of jQuery running on your site for this to work which can be found at jquery.com.

You will find a flexigrid.js file in the downloaded archive. Include this file in the head section of your site as you would normally do along with the provided CSS file (you will need to copy across the entire contents of the 'css' directory including the images).

After creating a table element on your page with an id of 'flex1' for this example you can then create and include a javascript file consisting of the following code. The Flexigrid will then be created on page load.

```
$(function() {
	$("#flex1").flexigrid({
		url: 'staff.php',
		dataType: 'json',
		colModel : [
			{display: 'ID', name : 'id', width : 40, sortable : true, align: 'left'},
			{display: 'First Name', name : 'first_name', width : 150, sortable : true, align: 'left'},
			{display: 'Surname', name : 'surname', width : 150, sortable : true, align: 'left'},
			{display: 'Position', name : 'email', width : 250, sortable : true, align: 'left'}
		],
		buttons : [
			{name: 'Edit', bclass: 'edit', onpress : doCommand},
			{name: 'Delete', bclass: 'delete', onpress : doCommand},
			{separator: true}
		],
		searchitems : [
			{display: 'First Name', name : 'first_name'},
			{display: 'Surname', name : 'surname', isdefault: true},
			{display: 'Position', name : 'position'}
		],
		sortname: "id",
		sortorder: "asc",
		usepager: true,
		title: "Staff",
		useRp: true,
		rp: 10,
		showTableToggleBtn: false,
		resizable: false,
		width: 700,
		height: 370,
		singleSelect: true
	});
});
```

## Properties ##

Using the above as a basis for your grid, you can customise it to suit your requirements using several javascript properties. For example, you can specify on which column the results should be sorted initially and whether to sort in ascending or descending order. The most common properties are shown below.

| **Property** | **Description** |
|:-------------|:----------------|
| url          | This is a URL of the server-side script which provides a JSON or XML representation of the results to display in the grid using AJAX. |
| dataType     | You can choose to have your server-side script return either JSON or XML data. |
| colModel     | This is an array containing a list of columns to use. You can set the display name, the width, whether the column can be sorted, and text alignment. |
| buttons      | It is possible using this array to add buttons along the top of the Flexigrid specifying a callback function, e.g. you may want to create buttons to add, edit, and delete records. The bClass property is the CSS class used to set the background image for the button, etc. |
| searchItems  | Using this you can specify which columns to use for searching the results using the 'quick search' area. You simply specify a display name for the search option, the column name, and whether the column is the default search option. |
| sortname     | This property is used to specify the initial column to sort on. |
| sortorder    | Specifying either 'asc' or 'desc' for this property will set the initial sort order. |
| usepager     | The page navigation buttons can be turned on or off using this property. |
| title        | This is the title which will appear at the top of the Flexigrid. |
| useRp        | Whether to allow the user to specify the number of results per page. |
| rp           | The initial number of results per page. |
| showTableToggleBtn | This will enable/disable minimisation of the Flexigrid with an icon in the top right corner. |
| width        | The width of the Flexigrid. |
| height       | The height of the Flexigrid. |
| singleSelect | This undocumented property is used to indicate that only one row can be selected at a time. This is useful if you would like to create buttons such as edit or delete which apply only to a single row. |

## The Server-side Script ##

For this example I will be using a PHP script to return the JSON results filtered by the criteria specified within the Flexigrid.

The following data is posted to our script using AJAX and can be found in PHP's $_POST array:_| **Parameter**  | **Description** |
|:---------------|:----------------|
| page           | Current page number. |
| sortname       |The name of the column to sort by. |
|sortorder       | The order to sort by – 'asc' or 'desc'. |
|qtype           | The column selected during 'quick search'. |
|query           | The text used within a search. |
|rp              | The number of records to be returned. |

Now that we have this information we can go ahead and create the script:
```
<?php
// Connect to MySQL database
mysql_connect('server', 'username', 'password');
mysql_select_db('dbname');
$page = 1; // The current page
$sortname = 'id'; // Sort column
$sortorder = 'asc'; // Sort order
$qtype = ''; // Search column
$query = ''; // Search string
// Get posted data
if (isset($_POST['page'])) {
	$page = mysql_real_escape_string($_POST['page']);
}
if (isset($_POST['sortname'])) {
	$sortname = mysql_real_escape_string($_POST['sortname']);
}
if (isset($_POST['sortorder'])) {
	$sortorder = mysql_real_escape_string($_POST['sortorder']);
}
if (isset($_POST['qtype'])) {
	$qtype = mysql_real_escape_string($_POST['qtype']);
}
if (isset($_POST['query'])) {
	$query = mysql_real_escape_string($_POST['query']);
}
if (isset($_POST['rp'])) {
	$rp = mysql_real_escape_string($_POST['rp']);
}
// Setup sort and search SQL using posted data
$sortSql = "order by $sortname $sortorder";
$searchSql = ($qtype != '' && $query != '') ? "where $qtype = '$query'" : '';
// Get total count of records
$sql = "select count(*)
from staff
$searchSql";
$result = mysql_query($sql);
$row = mysql_fetch_array($result);
$total = $row[0];
// Setup paging SQL
$pageStart = ($page-1)*$rp;
$limitSql = "limit $pageStart, $rp";
// Return JSON data
$data = array();
$data['page'] = $page;
$data['total'] = $total;
$data['rows'] = array();
$sql = "select id, first_name, surname, position
from staff
$searchSql
$sortSql
$limitSql";
$results = mysql_query($sql);
while ($row = mysql_fetch_assoc($results)) {
$data['rows'][] = array(
'id' => $row['id'],
'cell' => array($row['id'], $row['first_name'], $row['surname'], $row['position'])
);
}
echo json_encode($data);
?>
```

There's nothing too complicated here. We simply connect to a MySQL database (substitute the connection variables for the values corresponding to your own database), create the SQL from the data supplied by Flexigrid, get the number of records in the entire result set and return the results in JSON format using the json\_encode function which is available to PHP versions 5.2.0 and later.

Flexigrid requires a few parameters to be passed back in the JSON array. These are:
| **Parameter** | **Description** |
|:--------------|:----------------|
|page           | The current page number. |
|total          | The total number of records in the result set. Used by Flexigrid to calculate the number of pages. |
|rows           | This is an array containing the data for the rows. Each row needs a unique id (used within the id for the HTML 'tr' element) and an array of column data. |

## Buttons ##
In the javascript listing above we specified that the doCommand function should be called when either the 'edit' or 'delete' buttons are clicked on. How does Flexigrid know which button has been clicked on? Two parameters are passed to the function. These are the name of the command which in this case can be either 'Edit' or 'Delete', and the grid object.

All we are concerned with doing in this tutorial is finding an id for the selected row. When we have this we can easily perform whatever operation we wish on the data for the selected record using e.g. AJAX techniques which are beyond the scope of this tutorial.
```
function doCommand(com, grid) {
if (com == 'Edit') {
$('.trSelected', grid).each(function() {
var id = $(this).attr('id');
id = id.substring(id.lastIndexOf('row')+3);
alert("Edit row " + id);
});
} else if (com == 'Delete') {
$('.trSelected', grid).each(function() {
var id = $(this).attr('id');
id = id.substring(id.lastIndexOf('row')+3);
alert("Delete row " + id);
});
}
}
```
As you can see it is just a case of finding the selected row using jQuery. We then extract the numeric id of the record from the id of the HTML 'tr' element. You are free from this point on to implement the editing and adding functions however you'd like e.g. using jQuery UI dialogs.

_Tutorial originally assembled by the talented folks at http://www.kenthouse.com/_