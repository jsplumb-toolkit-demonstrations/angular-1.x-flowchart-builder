
<a name="top"></a>
## Flowchart Builder (Angular 1.x)

This is a port of the Flowchart Builder application that demonstrates the Toolkit's Angular 1.x integration.

![Flowchart Builder Demonstration](demo-flowchart.png)

This page gives you an in-depth look at how the application is put together.

<a name="package" ref="imports" title="package.json"></a>

```javascript
{
    "dependencies": {
        "font-awesome": "^4.7.0",
        "jsplumbtoolkit": "file:../../jsplumbtoolkit.tgz",
        "jsplumbtoolkit-angular-1.x":"file:../../jsplumbtoolkit-angular-1.x.tgz"
    }
}

```

[TOP](#top)

---

<a name="setup"></a>
### Page Setup

#### CSS

```xml
<link href="node_modules/font-awesome/css/font-awesome.min.css" rel="stylesheet">
<link rel="stylesheet" href="node_modules/jsplumbtoolkit/dist/css/jsplumbtoolkit-defaults.css">
<link rel="stylesheet" href="node_modules/jsplumbtoolkit/dist/css/jsplumbtoolkit-demo.css">
<link rel="stylesheet" href="app.css">
```
Font Awesome, `jsplumbtoolkit-demo.css`, and `app.css` are used for this demo and are not jsPlumb Toolkit requirements. `jsplumbtoolkit-defaults.css` is recommended for all apps using the Toolkit, at least when you first start to build your app. This stylesheet contains sane defaults for the various widgets in the Toolkit. 

#### JS

```xml
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.5/angular.js"></script>
<script src="node_modules/jsplumbtoolkit/dist/js/jsplumbtoolkit.js"></script>
<script src="node_modules/jsplumbtoolkit-angular-1.x/dist/js/jsplumbtoolkit-angular-1.x.js"></script>
<script src="app.js"></script>
```

We import `jsplumbtoolkit.js` and `jsplumbtoolkit-angular-1.x` from `node_modules` (they are listed in `package.json`). `app.js` contains the demo code; it is discussed on this page.

[TOP](#top)

---

<a name="templates"></a>
### Templates

There are four templates used by the app - one each for the node types of `Question`, `Action` and `Output`, and one 
for the `Start` node. These are inlined in `index.html`.

The templates look like this:

**Start**

```xml
<script type="text/ng-template" id="start_template.tpl">
    <div ng-attr-style="width:{% raw %}{{node.w}} {% endraw %}px;height:{% raw %}{{node.h}} {% endraw %}px;">
        <div style="position:relative">
            <svg ng-attr-width="{% raw %}{{ node.w }} {% endraw %}" ng-attr-height="{% raw %}{{ node.h }} {% endraw %}">
                <ellipse ng-attr-cx="{% raw %}{{ node.w/2 }} {% endraw %}" ng-attr-cy="{% raw %}{{ node.h/2 }} {% endraw %}" ng-attr-rx="{% raw %}{{ node.w/2 }} {% endraw %}" ng-attr-ry="{% raw %}{{ node.h/2 }} {% endraw %}" class="outer"/>
                <ellipse ng-attr-cx="{% raw %}{{ node.w/2 }} {% endraw %}" ng-attr-cy="{% raw %}{{ node.h/2 }} {% endraw %}" ng-attr-rx="{% raw %}{{ (node.w/2) - 10 }} {% endraw %}" ng-attr-ry="{% raw %}{{ (node.h/2) - 10 }} {% endraw %}" class="inner"/>
                <text text-anchor="middle" ng-attr-x="{% raw %}{{ node.w / 2 }} {% endraw %}" ng-attr-y="{% raw %}{{ node.h / 2 }} {% endraw %}" dominant-baseline="central">{% raw %}{{node.text}} {% endraw %}</text>
            </svg>
        </div>
    </div>
    <jtk-source port-type="start" filter=".outer" filter-negate="true"></jtk-source>
</script>
```

The **Start** node consists of an ellipse with a text label centered inside of it. Note here - and in all the other 
templates in this demo - there are a few main differences between the format of this template and that of a template using
the Toolkit's default templating mechanism:

- You do not need to prefix SVG elements with an `svg` namespace in Angular. 
- You must prefix SVG attributes if they contain an expression. There are numerous examples of this above: `ng-attr-cx`,
`ng-attr-cy` etc. This is how Angular handles the fact that SVG elements are unforgiving when an attribute does not have
a value that is in the correct format (due to the digest loop in Angular, values are not written when the template is
first rendered).
- You must also prefix a `style` attribute, eg `ng-attr-style` instead of just `style`. This is just for IE compatibility;
other browsers allow you to specify `style`.
- All references to the node data that is being rendered are prefixed with `node.`. For instance, the first line in the
template here is:

```xml
<div ng-attr-stylestyle="width:{% raw %}{{node.w}} {% endraw %}px;height:{% raw %}{{node.h}} {% endraw %}px;">
```

but in the version of this demo using the default templating mechanism it looks like this:

```xml
<div style="width:${w}px;height:${h}px;">
```

- Any `jtk-source`, `jtk-target` or `jtk-port` elements that are intended to configure the node's main container have to
be placed in the _root of the template_. This is in contrast to the default templating mechanism, which expects one
root element per template. It is due to the fact that the template is rendered by Angular inside of its directive element,
and the directive element is not discarded. There is a mechanism in Angular to instruct it to remove the original
directive element (`replace:true` in a directive), but this is deprecated and not recommended by Angular, and will not
work with the Toolkit integration.


The `jtk-source` element declares that this node is an edge source, of type `start` (the `port-type` attribute 
specifies this).  The `filter` attribute instructs the Toolkit to enable drag only from some element that is not a 
child of an `svg` element, but then `filter-negate` is `true`: the result is that dragging will begin only from a 
descendant of the `svg` element. What this means visually is that the user will not be able to start a drag from the 
whitespace surrounding the ellipse.

**Action**

```xml
<script type="text/ng-template" id="action_template.tpl">
    <div ng-attr-style="width:{% raw %}{{node.w}} {% endraw %}px;height:{% raw %}{{node.h}} {% endraw %}px;">
        <div style="position:relative">
            <div class="node-edit node-action" ng-click="editNode(node)">
                <i class="fa fa-pencil-square-o"></i>
            </div>
            <div class="node-delete node-action" ng-click="removeNode(node)">
                <i class="fa fa-times"></i>
            </div>
            <svg ng-attr-width="{% raw %}{{node.w}} {% endraw %}" ng-attr-height="{% raw %}{{node.h}} {% endraw %}">
                <rect x="0" y="0" ng-attr-width="{% raw %}{{node.w}} {% endraw %}" ng-attr-height="{% raw %}{{node.h}} {% endraw %}" class="outer"/>
                <rect x="10" y="10" ng-attr-width="{% raw %}{{node.w-20}} {% endraw %}" ng-attr-height="{% raw %}{{node.h-20}} {% endraw %}" class="inner"/>
                <text text-anchor="middle" ng-attr-x="{% raw %}{{node.w/2}} {% endraw %}" ng-attr-y="{% raw %}{{node.h/2}} {% endraw %}" dominant-baseline="central">{% raw %}{{node.text}} {% endraw %}</text>
                <svg>
        </div>
        <jtk-target port-type="target"></jtk-target>
        <jtk-source port-type="source" filter=".outer"></jtk-source>
    </div>
</script>
```

Once again we use the position and dimensions for the node's main container as well as its SVG elements. **Action** 
nodes are configured as both edge sources and targets.

**Question**

```xml
<script type="text/ng-template" id="question_template.tpl">
    <div ng-attr-style="width:{% raw %}{{node.w}} {% endraw %}px;height:{% raw %}{{node.h}} {% endraw %}px;">
        <div class="node-edit node-action" ng-click="editNode(node)">
            <i class="fa fa-pencil-square-o"></i>
        </div>
        <div class="node-delete node-action" ng-click="removeNode(node)">
            <i class="fa fa-times"></i>
        </div>
        <svg ng-attr-width="{% raw %}{{node.w}} {% endraw %}" ng-attr-height="{% raw %}{{node.h}} {% endraw %}">
            <path ng-attr-d="M {% raw %}{{node.w/2}} {% endraw %} 0 L {% raw %}{{node.w}} {% endraw %} {% raw %}{{node.h/2}} {% endraw %} L {% raw %}{{node.w/2}} {% endraw %} {% raw %}{{node.h}} {% endraw %} L 0 {% raw %}{{node.h/2}} {% endraw %} Z" class="outer"/>
            <path ng-attr-d="M {% raw %}{{node.w/2}} {% endraw %} 10 L {% raw %}{{node.w-10}} {% endraw %} {% raw %}{{node.h/2}} {% endraw %} L {% raw %}{{node.w/2}} {% endraw %} {% raw %}{{node.h-10}} {% endraw %} L 10 {% raw %}{{node.h/2}} {% endraw %} Z" class="inner"/>
            <text text-anchor="middle" ng-attr-x="{% raw %}{{node.w/2}} {% endraw %}" ng-attr-y="{% raw %}{{node.h/2}} {% endraw %}" dominant-baseline="central">{% raw %}{{node.text}} {% endraw %}</text>
        </svg>
    </div>
    <jtk-source port-type="source" filter=".outer"></jtk-source>
    <jtk-target port-type="target"></jtk-target>
</script>
```

The **Question** node draws a diamond, and declares itself to be an edge target and source. 

**Output**

```xml
 <script type="text/ng-template" id="output_template.tpl">
     <div ng-attr-style="width:{% raw %}{{node.w}} {% endraw %}px;height:{% raw %}{{node.h}} {% endraw %}px;">
         <div style="position:relative">
             <div class="node-edit node-action" ng-click="editNode(node)">
                 <i class="fa fa-pencil-square-o"></i>
             </div>
             <div class="node-delete node-action" ng-click="removeNode(node)">
                 <i class="fa fa-times"></i>
             </div>
             <svg ng-attr-width="{% raw %}{{node.w}} {% endraw %}" ng-attr-height="{% raw %}{{node.h}} {% endraw %}">
                 <rect x="0" y="0" ng-attr-width="{% raw %}{{node.w}} {% endraw %}" ng-attr-height="{% raw %}{{node.h}} {% endraw %}"/>
                 <text text-anchor="middle" ng-attr-x="{% raw %}{{node.w/2}} {% endraw %}" ng-attr-y="{% raw %}{{node.h/2}} {% endraw %}" dominant-baseline="central">{% raw %}{{node.text}} {% endraw %}</text>
             </svg>
         </div>
     </div>
     <jtk-target port-type="target"></jtk-target>
 </script>
```

The `Output` node is configured to be a connection target only.

[TOP](#top)

---

<a name="directives"></a>
### Directive Configuration

Everything is contained within one element that declares the controller and app to use:

```xml
<div class="jtk-demo-main" id="jtk-demo-flowchart" ng-controller="DemoController as DemoController" ng-app="app">
  ...
</div>
```

The demo uses three directives. Here we see how the Toolkit and its Miniview are declared:

```xml
<jsplumb-toolkit params="DemoController.toolkitParams"
         init="DemoController.init"
         render-params="DemoController.renderParams"
         jtk-id="flowchartToolkit"
         surface-id="flowchartSurface"
         style="width:750px;height:600px;position:relative;margin-right: 20px;">

    ...

    <!-- miniview -->
    <jsplumb-miniview surface-id="flowchartSurface" class="miniview"></jsplumb-miniview>

</jsplumb-toolkit>
```

Points to note:

- Both `params` (for the Toolkit constructor) and `renderParams` (for the renderer) are provided.
- An ID is assigned to the Toolkit and to the surface
- An `init` method is declared; this will be executed after the Toolkit and Surface have been created.


Here is how the palette (the set of nodes that can be dragged into the work area) is declared:

```xml
<div class="sidebar node-palette" jsplumb-palette selector="li" surface-id="flowchartSurface" type-extractor="DemoController.typeExtractor">
    <ul ng-repeat="node in nodeTypes">
        <li jtk-node-type="{% raw %}{{ node.type }} {% endraw %}" title="Drag to add new">
            {% raw %}{{node.label}} {% endraw %}
        </li>
    </ul>
</div>
```

Points to note:

- `nodeTypes` is an object in the scope of `DemoController`.
- `surface-id` maps to the `surface-id` specified in the `jsplumb-toolkit` directive.
- The `selector` attribute instructs the Palette to look for any `li` elements that are descendants of the Palette's element.
- A `type-extractor` function is specified; this is what determines the type of newly dropped elements.


[TOP](#top)

---

<a name="initialisation"></a>
### Initialisation

`DemoController.init` looks like this:

```javascript
function(scope, element, attrs) {

    toolkit = scope.toolkit;

    toolkit.load({
        url:"data/flowchart-1.json"
    });

    // -------------- configure buttons --------------------------------

    var controls = element[0].querySelector(".controls");
    // listener for mode change on renderer.
    scope.surface.bind("modeChanged", function (mode) {
        jsPlumb.removeClass(controls.querySelectorAll("[mode]"), "selected-mode");
        jsPlumb.addClass(controls.querySelectorAll("[mode='" + mode + "']"), "selected-mode");
    });

    // pan mode/select mode
    jsPlumb.on(controls, "tap", "[mode]", function () {
        scope.surface.setMode(this.getAttribute("mode"));
    });

    // on home button click, zoom content to fit.
    jsPlumb.on(controls, "tap", "[reset]", function () {
        scope.toolkit.clearSelection();
        scope.surface.zoomToFit();
    });

    // configure Drawing tools.
    new jsPlumbToolkit.DrawingTools({
        renderer: scope.surface
    });
};

```

The main things to note here are that the `scope` passed in to this method contains `toolkit` and `surface` members. This demo assigns the `toolkit` member to a variable in the Controller's scope, so it is subsequently available everywhere.
 
[TOP](#top)

---

<a name="loading"></a>
### Data Loading

Data for this application is stored in `data/flowchart-1.json` inside the application folder. It is loaded by this code inside the `init` function:

```javascript
toolkit.load({
    url:"data/flowchart-1.json"
});
```

[TOP](#top)

---

<a name="dragging"></a>
### Dragging new nodes

As discussed above, a `jsplumb-palette` is declared, which configures all of its child `li` elements to be droppable onto the Surface canvas.  When a drop occurs, the type of the newly dragged node is calculated by the `typeExtractor` function declared on `DemoController`:

```javascript
this.typeExtractor = function (el) {
    return el.getAttribute("jtk-node-type");
};
```

[TOP](#top)

---

<a name="behaviour"></a>
### Behaviour

In the original FlowchartBuilder demo, there are three pieces of behaviour that we need to code that are not completely handled for us by the Toolkit:  

- Edit Node Text
- Delete Node
- Remove Edge

These exist in this Angular demo, too, but, for Nodes, are handled slightly differently:

###### Edit Node Text

```xml
<div class="node-edit node-action" ng-click="editNode(node)">
    <i class="fa fa-pencil-square-o"></i>
</div>
```

```javascript
$scope.editNode = function (node) {    
    var info = this.surface.getObjectInfo(node);
    jsPlumbToolkit.Dialogs.show({
        id: "dlgText",
        data: info.obj,
        title: "Edit " + info.obj.type + " name",
        onOK: function (data) {
            if (data.text && data.text.length > 2) {
                // if name is at least 2 chars long, update the underlying data and
                // update the UI.
                toolkit.updateNode(info.obj, data);
            }
        }
    });
};
```

The meat of this method is the same as that which the original FlowchartBuilder demo has; it's just the trigger to launch the method that is different.

Note here the `getObjectInfo` method: this is method with which you will want to acquaint yourself. It is defined on a Surface, not on a Toolkit instance, and takes a DOM element as argument and places it within the context of some object managed by the Toolkit instance that the Surface is associated with. In this case, the click event occurs on an icon. `getObjectInfo` traverses up the icon's ancestors until it finds an element that is associated with a Toolkit object - in our app, either a Table or View. When a Toolkit object is found, getObjectInfo returns an object with these values:

- **id** ID of the Toolkit object
- **obj** The associated Toolkit object. May be a Node or a Port.
- **el** The DOM element for the Toolkit object
- **type** The type of the Toolkit object. This will be "Node" or "Port".

In this event handler, we show the user a dialog that will allow them to edit the Node's text. If the edited text is at least two character long we update the model.

This application uses the Toolkit's [dialogs](https://docs.jsplumbtoolkit.com/toolkit/current/articles/dialogs) import to manage simple interactions with data members such as this. Your application may choose to use a different mechanism.

###### Delete Node

```xml
<div class="node-delete node-action" ng-click="removeNode(node)">
    <i class="fa fa-times"></i>
</div>
```

###### Remove Edge

This is handled in the same way by this demo as it is in the original FlowchartBuider.

We register a `dblclick` listener on edges by providing it as an event handler to the View, on the edge type that acts as the parent type for all others:

```javascript
...
edges: {
  "default": {
    connector: ["Flowchart", { cornerRadius: 5 } ],
    paintStyle: { lineWidth: 2, strokeStyle: "#f76258", outlineWidth: 3, outlineColor: "transparent" },	
    hoverPaintStyle: { lineWidth: 2, strokeStyle: "rgb(67,67,67)" }, 
    events: {
      "dblclick": function (params) {
        jsPlumbToolkit.Dialogs.show({
          id: "dlgConfirm",
          data: {
            msg: "Delete Edge"
          },
          onOK: function () {
            toolkit.removeEdge(params.edge);
          }
        });
      }
   },

...
```

###### Editing Edge Label

All Edges except those from a *Start* node are set to be of type `connection`, which is defined in the view as follows:

```javascript
"connection":{
  parent:"default",
  overlays:[
    [ "Label", {
      label: "${label}",
      events:{
        click:function(params) {
          _editLabel(params.edge);
        }
      }
    }]
  ]
}
```

We define a `click` event handler on the Label overlay which edits the label for the Edge.

[TOP](#top)

---

<a name="resizedrag"></a>
### Resizing/Dragging Nodes

To resize or drag a node first you must either click on it, or use the lasso (described [below](#selecting)) to select it.  A selected node looks like this:

![Selected Node](flowchart-selected-node.png)

The dotted line and drag handles that are added to a selected Node are put there by the Tookit's drawing tools. It listens to the Toolkit's select/deselect events and decorates UI elements accordingly. These tools are discussed in detail on [this page](https://docs.jsplumbtoolkit.com/toolkit/current/articles/drawing).

The drawing tools are initialized with this line of code (inside `DemoController.init`):

```javascript
// configure Drawing tools.
new jsPlumbToolkit.DrawingTools({renderer: renderer});
```

You pass them the instance of the Surface widget you're working with.

#### Dragging

Nodes can be dragged only by the square in the center of the node. This is achieved by setting a `filter` on the 
`dragOptions` parameter on the `render` call:

```javascript
dragOptions: {
  handle: ".jtk-draw-drag"
}
```

`jtk-draw-drag` is the classname of the square that the drawing tools place in the center of a selected node.

#### Resizing

Resizing is handled automatically by the drawing tools.  By default, these tools will change the `w`, `h`, `left` and `top` values in a node's data, but this can be changed.

When a node's data is updated the drawing tools call the appropriate update method on the underlying Toolkit. The changes will be reflected immediately in the DOM.

[TOP](#top)

---

<a name="selecting"></a>
### Selecting Nodes

#### Left Click

Nodes can be selected with a left-click (or tap on a touch device; `tap` is a better event to choose in general because the
Toolkit abstracts out the difference between mouse devices and touch devices and presents `click` events as `tap` events
on non touch devices).  This is configured in the `view` parameter to the `render` call. In this application, 
Nodes of type _selectable_ have the capability enabled with this code:

```javascript
"selectable": {
  events: {
    tap: function (params) {
      toolkit.toggleSelection(params.node);
    }
  }
}
```

The `tap` event (discussed [here](https://docs.jsplumbtoolkit.com/toolkit/current/articles/widget-surface#eventlist)) is preferable to `click`, as it ensures the application responds only to true clicks on devices with a mouse, and also avoids the delay that some vendors introduce to a click event on touch devices.

#### Lasso Selection

Lasso selection is enabled by default on the Surface widget. 

To activate the lasso, click the pencil icon in the toolbar:

![Lasso Select Mode](select-lasso.png)

The code that listens to clicks on this icon is as follows:

```javascript
// listener for mode change on renderer.
renderer.bind("modeChanged", function (mode) {
  jsPlumb.removeClass(jsPlumb.getSelector("[mode]"), "selected-mode");
  jsPlumb.addClass(jsPlumb.getSelector("[mode='" + mode + "']"), "selected-mode");
});

// pan mode/select mode
jsPlumb.on(".controls", "click", "[mode]", function () {
  renderer.setMode(this.getAttribute("mode"));
});
```

Here we've registered an event listener to be informed when the Surface's mode has changed; it is responsible for setting the appropriate classes on the toolbar items.  The click listener extracts the desired mode from the button that was clicked and sets it on the renderer.

##### Lasso Operation

The lasso works in two ways: when you drag from left to right, any node that intersects your lasso will be selected.  When you drag from right to left, only nodes that are enclosed by your lasso will be selected.

![Lasso Operation](lasso-select-operation.png)

##### Exiting Select Mode

The Surface widget automatically exits select mode once the user has selected something. In this application we also listen to clicks on the whitespace in the widget and switch back to pan mode when we detect one. This is the `events` argument to the `render` call:

```javascript
events: {
  canvasClick: function (e) {
    toolkit.clearSelection();
  }
}
```

`clearSelection` clears the current selection and switches back to Pan mode.

[TOP](#top)

---

<a name="dialogs"></a>
### Dialogs

The dialogs used in this app are part of the jsPlumb Toolkit core. They provide a simple abstraction around the business of getting input from the user and dealing with it; they're not necessarily fully-featured enough for all applications.

#### Initialization

To initialize the dialogs, first call `jsPlumbToolkit.Dialogs.initialize`, with an appropriate selector for the templates for your dialogs (see below for an explanation of this):

```javascript
jsPlumbToolkit.Dialogs.initialize({
  selector:".dlg"
});

var showDialog = jsPlumbToolkit.Dialogs.show;
```

Note here we've also aliased `jsPlumbToolkit.Dialogs.show`, to save some typing.

#### Templates

Each dialog has a template in the HTML, with some class name that you matched in the `selector` argument to the `initialize` call above:

```xml
<script type="jtk" class="dlg" id="dlgViewQuery" title="Edit Query">
  <textarea class="txtViewQuery" jtk-focus jtk-att="query">${query}</textarea>
</script>
```

##### Binding Parameters

These templates use the same template engine as the Surface renderer, so in this example you can see we've extracted `query` from the View node's data, and injected it into the textarea. But what might not be immediately obvious is the purpose of the `jtk-att` attribute: it tells the dialog handler that the value of this textarea should be passed to the OK handler, using the key `query`.

Note also in the above example, the `jtk-focus` attribute: this tells the dialog handler that the textarea should be given the focus when the dialog first opens.

#### Showing a dialog

This example is the dialog that is shown when you edit a View query. We provide the id of the template to use for the dialog, and we provide the View node's data as the backing data for the dialog. Then we provide an `onOK` callback:

```javascript
jsPlumbToolkit.Dialogs.show({
  id:"dlgViewQuery",
  data:info.obj.data,
  onOK:function(data) {
    // update data
    toolkit.updateNode(info.obj, data);
    // update UI	        		
    info.el.querySelectorAll(".view-details")[0].innerHTML = data.query;
  }
});
```

The `data` argument to the `onOK` callback consists of an object whose key value pairs are determined by the `jtk-att` attributes found in the template. Recall that above we had a textarea with `jtk-att:"query"`. This means that the `data` argument to `onOK` looks like this:

```javascript
{
  query:"the contents of the text area"
}
```

##### Supported Input Types

The list of supported input types is:

- text
- radio button(s) 
- checkbox
- select
- textarea

##### Dialog Titles

If you set a `title` attribute on a dialog's template, that value will be used for the title of the dialog. Alternatively, you can provide a `title` parameter to the `show` call.

##### Lifecycle Callbacks

There are three lifecycle callbacks supported:

- **onOpen** Called when the dialog has been opened. The related DOM element for the dialog is passed to this method.
- **onOK** Called when the user presses OK. The callback is provided a data object as discussed above.
- **onCancel** Called when the user presses cancel. No arguments are provided to the callback.

[TOP](#top)
