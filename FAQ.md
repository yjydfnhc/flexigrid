# Frequently Asked Questions #

## Typical Grid and Debug tools ##

In these examples, we use a typical Grid:
```
var myFlex = { 
    autoload    : false, 
    dataType    : 'json', 
    url         : 'grid.php', 
    usepager    : true 
}; 

$('#grid01').flexigrid($.extend({}, myFlex, { 
    colModel    : [ 
        {name: 'name',   display: 'NAME',   sortable: true}, 
        {name: 'status', display: 'STATUS', sortable: true}, 
        {name: 'sign',   display: 'SIGN',   sortable: true} 
    ]
})); 
```

For debug we use [console.log()](http://getfirebug.com/logging) more likely than alert(), which is not always work as expected.

## Examples ##

  * [#Dynamic\_columns\_and\_Post\_parameter\_to\_server](#Dynamic_columns_and_Post_parameter_to_server.md)
  * [#Get\_selected\_row](#Get_selected_row.md)
  * [#Mouseclick\_and\_other\_events](#Mouseclick_and_other_events.md)
  * [#Multiple\_filters](#Multiple_filters.md)
  * [#Format\_cells](#Format_cells.md)

### Dynamic columns and Post parameter to server ###

Colmodel define in variable.
Parameters send to server by .flexOptions() on onSubmit().

```
var colModel01 = [ 
        {name: 'name',   display: 'NAME',   sortable: true}, 
        {name: 'status', display: 'STATUS', sortable: true}, 
        {name: 'sign',   display: 'SIGN',   sortable: true} 
]; 

$('#grid01').flexigrid($.extend({}, myFlex, { 
    colModel    : colModel01, 
    onSubmit    : function(){ 
        $('#grid01').flexOptions({params: [ 
            {name:'colls', value: $.param({colls: $.map(colModel01, function(elem,id){return elem.name}) }) } 
        ]}); 
        return true; 
    } 
})); 
```

In this code we post to server on load next data (with column names):
```
page=1&rp=15&sortname=undefined&sortorder=undefined&query=&qtype=& 
colls=colls%255B%255D%3Dname%26colls%255B%255D%3Dstatus%26colls%255B 
%255D%3Dsign 
```

### Get selected row ###

The difficulty lies in linking: column\_name - data. That is why the order of columns can be changed, and we can not use index in the array of row cells.
In flexigrid-1.1 we have the attribute 'abbr' for sortable table cells ([issue 46](https://code.google.com/p/flexigrid/issues/detail?id=46)).

```
$('#grid01').click(function(event){
    $('.trSelected', this).each( function(){
        console.log(
            '  rowId: '  + $(this).attr('id').substr(3) +
            '  name: '   + $('td[abbr="name"] >div', this).html() +
            '  sign: '   + $('td[abbr="sign"] >div', this).html() +
            '  status: ' + $('td[abbr="status"] >div', this).html() 
        );
    });
});
```

### Mouseclick and other events ###

As of jQuery 1.7, the [.on()](http://api.jquery.com/on/) method provides all functionality required for attaching event handlers.

```
$('.flexigrid').on('mouseenter', 'tr[id*="row"]', function(){
    console.log('mouseenter rowId: ' + $(this).attr('id').substr(3));
});
```

With older jQuery:
If we bind the event to the cells, we have to use the [.live()](http://api.jquery.com/live/) or [.delegate()](http://api.jquery.com/delegate/). Because it when receiving data grid content is recreated.

```
$('.flexigrid tr[id*="row"]').live('mouseenter', function(){
    console.log('mouseenter rowId: ' + $(this).attr('id').substr(3));
});
```

### Multiple filters ###

Just add a custom filter form
```
<form id="fmFilter"> 
    <input id="fmFilterSel1" name="fmFilterSel1" type="checkbox" /> 
    <input id="fmFilterSel2" name="fmFilterSel2" type="checkbox" /> 
    <input id="fmFilterSel3" name="fmFilterSel3" type="text" /> 
</form> 
<table id="grid01"><tr><td></td></tr></table> 

<script> 
$('#grid01').flexigrid($.extend({}, myFlex, { 
    onSubmit : function(){
        $('#grid01').flexOptions({params: [{name:'callId', value:'grid01'}].concat($('#fmFilter').serializeArray())});
        return true;
    } 
}); 
</script> 
```
And you send to server the filter condition of any complexity.

### Format cells ###

You can colorize the cells, and even change the output according to the received data.

```
function gridFormat() { 
    var lblStatus = { 
        '101' : { 
            css : '', 
            txt : 'STATUS_OK' 
        }, 
        '102' : { 
            css : 'cellDisable', 
            txt : 'STATUS_LOCK' 
        }, 
        '103' : { 
            css : 'cellWarning', 
            txt : 'STATUS_BAD' 
        } 
    }; 
    $('#grid01 tr').each( function(){ 
        var cell = $('td[abbr="status"] >div', this); 
        $(this).addClass( lblStatus[cell.text()].css ); 
        cell.text( lblStatus[cell.text()].txt ); 
    }); 
    return true; 
}

$('#grid01').flexigrid($.extend({}, myFlex, { 
    buttons     : [
        {name: 'CLEAR', onpress: function(com,grid){ $('#grid01').flexAddData({rows:[],page:0,total:0}); }},
        {name: 'FILL',  onpress: function(com,grid){ $('#grid01').flexAddData({rows:[
            {id:'id0',cell:['name00',101,'A']},
            {id:'id1',cell:['name01',102,'B']},
            {id:'id2',cell:['name02',103,'C']}
        ],page:1,total:3}); }}
    ],
    onSuccess   : gridFormat 
}); 
```

In this example, the Grid has a buttons for fill/clear the test data.
In column "status" we provide text labels for numeric values.
Keep in mind that 'abbr' implemented in sortable columns only ([#Get\_selected\_row](#Get_selected_row.md)).