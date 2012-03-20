# Envision.js
-------------

Fast interactive HTML5 charts.

![Google Groups](http://groups.google.com/intl/en/images/logos/groups_logo_sm.gif)

http://groups.google.com/group/envisionjs/

## Features

* Modern Browsers, IE 6+
* Mobile / Touch Support
* Pre-built Templates
* Adaptable to Existing Libraries

## Dependencies

Envision.js ships with all it's dependencies.  It uses:

* <a href="http://documentcloud.github.com/underscore/">underscore.js</a>
* <a href="https://github.com/fat/bean">bean</a>
* <a href="https://github.com/ded/bonzo">bonzo</a>
* <a href="http://humblesoftware.com/flotr2/">Flotr2</a>

## Usage

To use Envision.js, include `envision.min.js` and `envision.min.css` in your
page. To display a visualization, either use a Template or create a custom
visualization with the Envision.js API.

### Templates

Templates are pre-built visualizations for common use-cases.

Example: 

```javascript
  var
    container = document.getElementById('container'),
    x = [],
    y1 = [],
    y2 = [],
    data, options, i;

  // Data Format:
  data = [
    [x, y1], // First Series
    [x, y2]  // Second Series
  ];

  // Sample the sine function for data
  for (i = 0; i < 4 * Math.PI; i += 0.05) {
    x.push(i);
    y1.push(Math.sin(i));
    y2.push(Math.sin(i + Math.PI));
  }

  // TimeSeries Template Options
  options = {
    // Container to render inside of
    container : container,
    // Data for detail (top chart) and summary (bottom chart)
    data : {
      detail : data,
      summary : data
    }
  };

  // Create the TimeSeries
  new envision.templates.TimeSeries(options);
```

### Custom

Developers can use the envision APIs to build custom visualizations.  The
existing templates are a good reference for this.

Example: 

```javascript
  var
    container = document.getElementById('container'),
    x = [],
    y1 = [],
    y2 = [],
    data, i,
    detail, detailOptions,
    summary, summaryOptions,
    vis, selection,

  // Data Format:
  data = [
    [x, y1], // First Series
    [x, y2]  // Second Series
  ];

  // Sample the sine function for data
  for (i = 0; i < 4 * Math.PI; i += 0.05) {
    x.push(i);
    y1.push(Math.sin(i));
    y2.push(Math.sin(i + Math.PI));
  }
  x.push(4 * Math.PI)
  y1.push(Math.sin(4 * Math.PI));
  y2.push(Math.sin(4 * Math.PI));

  // Configuration for detail:
  detailOptions = {
    name : 'detail',
    data : data,
    height : 150,
    flotr : {
      yaxis : {
        min : -1.1,
        max : 1.1
      }
    }
  };

  // Configuration for summary:
  summaryOptions = {
    name : 'summary',
    data : data,
    height : 150,
    flotr : {
      yaxis : {
        min : -1.1,
        max : 1.1
      },
      selection : {
        mode : 'x'
      }
    }
  };

  // Building a custom vis:
  vis = new envision.Visualization();
  detail = new envision.Component(detailOptions);
  summary = new envision.Component(summaryOptions);
  interaction = new envision.Interaction();

  // Render Visualization
  vis
    .add(detail)
    .add(summary)
    .render(container);

  // Wireup Interaction
  interaction
    .leader(summary)
    .follower(detail)
    .add(envision.actions.selection);
```

## API

### Class `envision.Component`
_Defines a visualization component._

Components are the building blocks of a visualization, 
representing one typically graphical piece of the vis.  This class manages
the options, DOM and API construction for an adapter which handles the
actual drawing of the visualization piece.

Adapters can take the form of an actual object, a constructor function
or a function returning an object.  Only one of these will be used.  If
none is submitted, the default adapter Flotr2 is used.

#### Configuration:

An object is submitted to the constructor for configuration.

* `name` A name for the component.
* `element` A container element for the component.
* `height` An explicit component height.
* `width` An explicit component width.
* `data` An array of data.  Data may be formatted for 
envision or for the adapter itself, in which case skipPreprocess will
also need to be submitted.
* `skipPreprocess` Skip data preprocessing.  This is useful
when using the native data format for an adapter.
* `adapter` An adapter object.
* `adapterConstructor` An adapter constructor to be
instantiated by the component.
* `adapterCallback` An callback invoked by the component
returning an adapter.
* `adapterOptions` Options passed to the adapter constructor
or function.

#### Methods:

##### `render ([element])`
Render the component.

If no element is submitted, the component will
render in the element configured in the constructor.

##### `draw ([data], [options])`
Draw the component.

##### `trigger ()`
Trigger an event on the component's API.

Arguments are passed through to the API.

##### `attach ()`
Attach to an event on the component's API.

Arguments are passed through to the API.

##### `detach ()`
Detach a listener from an event on the component's API.

Arguments are passed through to the API.

##### `destroy ()`
Destroy the component.

Empties the container and calls the destroy method on the
component's API.

### Class `envision.Visualization`
_Defines a visualization of componenents._

This class manages the rendering of a visualization.
It provides convenience methods for adding, removing, and reordered
components dynamically as well as convenience methods for working
with a logical group of components.

#### Configuration:

An object is submitted to the constructor for configuration.

* `name` A name for the visualization.
* `element` A container element for the visualization.

#### Methods:

##### `render ([element])`
Render the visualization.

If no element is submitted, the visualization will
render in the element configured in the constructor.

This method is chainable.

##### `add (component)`
Add a component to the visualization.

If the visualization has already been rendered,
it will render the new component.

This method is chainable.

##### `remove ()`
Remove a component from the visualization.

This removes the components from the list of components in the
visualization and removes its container from the DOM.  It does not
destroy the component.

This method is chainable.

##### `setPosition (component, newIndex)`
Reorders a component.

This method is chainable.

##### `indexOf (component)`
Gets the position of a component.

##### `getComponent (component)`
Gets the component at a position.

##### `isFirst (component)`
Gets whether or not the component is the first component
in the visualization.

##### `isLast (component)`
Gets whether or not the component is the last component
in the visualization.

##### `destroy ()`
Destroys the visualization.

This empties the container and destroys all the components which are part
of the visualization.

### Class `envision.Preprocessor`
_Data preprocessor._

Data can be preprocessed before it is rendered by an adapter.

This has several important performance considerations.  If data will be 
rendered repeatedly or on slower browsers, it will be faster after being
optimized.

First, data outside the boundaries does not need to be rendered.  Second,
the resolution of the data only needs to be at most the number of pixels
in the width of the visualization.

Performing these optimizations will limit memory overhead, important
for garbage collection and performance on old browsers, as well as drawing
overhead, important for mobile devices, old browsers and large data sets.

#### Configuration:

An object is submitted to the constructor for configuration.

* `data` The data for processing.

#### Methods:

##### `getData ()`
Returns data.

##### `setData ()`
Set the data object.

##### `length ()`
Returns the length of the data set.

##### `bound (min, max)`
Bounds the data set at within a range.

##### `subsampleMinMax (resolution)`
Subsample data using MinMax.

MinMax will display the extrema of the subsample intervals.  This is
slower than regular interval subsampling but necessary for data that 
is very non-homogenous.

##### `subsample (resolution)`
Subsample data at a regular interval for resolution.

This is the fastest subsampling and good for monotonic data and fairly
homogenous data (not a lot of up and down).

### Class `envision.Interaction`
_Defines an interaction between components._

This class defines interactions in which actions are triggered
by leader components and reacted to by follower components.  These actions
are defined as configurable mappings of trigger events and event consumers.
It is up to the adapter to implement the triggers and consumers.

A component may be both a leader and a follower.  A leader which is a 
follower will react to actions triggered by other leaders, but will safely
not react to its own.  This allows for groups of components to perform a
common action.

Optionally, actions may be supplied with a callback executed before the 
action is consumed.  This allows for quick custom functionality to be added
and is how advanced data management (ie. live Ajax data) may be implemented.

This class follow an observer mediator pattern.

#### Configuration:

An object is submitted to the constructor for configuration.

* `leader` Component(s) to lead the
interaction

#### Methods:

##### `leader (component)`
Add a component as an interaction leader.

##### `follower (component)`
Add a component as an interaction leader.

##### `group (components)`
Adds an array of components as both followers and leaders.

##### `add (action, [options])`
Adds an action to the interaction.

The action may be optionally configured with the options argument.
Currently the accepts a callback member, invoked after an action
is triggered and before it is consumed by followers.

## Development

This project uses [smoosh](https://github.com/fat/smoosh) to build and [jasmine](http://pivotal.github.com/jasmine/) 
with [js-imagediff](https://github.com/HumbleSoftware/js-imagediff) to test.  Tests may be executed by 
[jasmine-headless-webkit](http://johnbintz.github.com/jasmine-headless-webkit/) with 
`cd spec; jasmine-headless-webkit -j jasmine.yml -c` or by a browser by navigating to 
`spec/SpecRunner.html`.

