Multiple Data Layers and Parallel Coordinate Plots
==================================================

This demo covers some methods for handling multiple data layers, displaying them on a map, and creating a Parallel Coordinate Plot (PCP) that corresponds to a data layer. 

## Multiple Data Layers

As you start getting further into your final projects you're probably noticing a couple of things. First, that there's a lot of data you want to interact with. Second, that all of this data is stored in different files. 

This structure is advantageous in that it breaks data down into nice little layers which you can easily style. But to this point we have loaded data files asynchronously, which is great until you have two layers that rely on each other in some way. 

May I introduce a library called [Queue.js](https://github.com/mbostock/queue). This is a simple library that you can use to complete a series of async tasks and then waiting for them all to finish before doing the next step. Here's a basic overview of how to use the library, and its API. 

### API

<a name="queue" href="queue">#</a> **queue**(_parallelism?_)

Creates a new queue that you can add tasks to. 

The _parallelism_ parameter is an optional Number that tells the library how many tasks can be completed at any one time. This defaults to 1, but you can increase the value to something like 3 to see improved performance. 

<a name="differ" href="differ">#</a> queue.**differ**(_task_, _parameters?..._)

Adds a tasks to the queue, in essence telling the application to differ progress until completing that task. 

The _task_ parameter is required and **must** be a function. The function can take any number of parameters you want, but the last parameter is a callback function. 

This callback function takes two parameters, _error_ and _result_. The _error_ parameter contains any errors that occurred during the task, and the result contains the result of the task. You **must** call the callback function in your code in order for this o work properly.

<a name="await" href="await">#</a> queue.**await**(_callback_)

Defines the function to be called when all tasks in the queue have been completed. 

The _callback_ function is required and takes an unspecified number of parameters. The first parameter is error, defined to the the first error encountered while completing the tasks in the queue. The rest of the parameters are the results of the various tasks in the queue. Therefore, if you added three tasks to the queue, the function you pass to **await**() will have four parameters, and error and three results. 

### Example Usage

```HTML 

<html lang="en">
<head>
  <script src="//d3.v3.min.js" charset="utf-8"></script>
  <script src="//queue.js" charset="utf-8"></script>
</head>
<body>
  <script type="text/javascript">
    function request (url, callback) {
        d3.json(url, function (error, data) {
            callback(error, data);
        })
    }

    queue(2)
      .differ(request, 'file1.json')
      .differ(request, 'file2.json')
      .await(function (error, file1, file2) {
        if (error) {
            return console.warn(error);
        }

        // Do some stuff with the data...
      });
  </script>
</body>
</html>

```

## Creating a Parallel Coordinate Plot

Many of your have expressed a desire to create a PCP from your data. So, I decided it was about time to make a simple plug-in for D3.js that will handle all the heavy lifting for you. I give you, [pcpGeoJson](https://github.com/RyanMullins/pcpGeoJson). Below is an discussion of the API and an example usage.

### API

<a name="d3_pcpgeojson" href="#d3_pcpgeojson">#</a> d3.**pcpGeoJson**(_collection_[, _options_]);

Creates a PCP for the properties, with values of type _Number_, of each [Features][geojsonf] in a [Feature Collection][geojsonfc]. 

The *collection* parameter is required and **must** be a GeoJSON Feature collection. Behavior is undefined if this parameter is not a Feature Collection. 

The *options* parameter is an optional object that allows you to define the following properties:

* _id_      : [String] The ID of the container for the PCP, defaults to 'body'
* _keys_    : [Array] List of all property names to be used as dimensions of the PCP, defaults to all properties in the first feature that has data of type Number
* _buffers_ : [Array] Array of numbers, must specify all; [top, right, bottom, left], defaults to [30, 10, 10, 10]
* _width_   : [Number] Width of the plot, defaults to 960
* _height_  : [Number] Height of the plot, defaults to 500 
* _linker_  : [Function] Callback taking a single parameter, passed a D3 selection object containing all foreground lines representing the features in the collection.

### Example Usage

```HTML 

<html lang="en">
<head>
  <script src="//d3.v3.min.js" charset="utf-8"></script>
  <script src="//d3.pcpGeoJson.min.js"></script>
  <link href="//d3.pcpGeoJson.css" rel='stylesheet' />
</head>
<body>
  <script type="text/javascript">
    d3.json('collection.json', function (error, collection) {
      if (error) {
        return console.warn(error);
      }

      d3.pcpGeoJson(collection);
    })
  </script>
</body>
</html>

```

## A More Complete Example

I have included a more complete example of these libraries with this demo. The _map.html_ file builds on the [Animation](https://github.com/RyanMullins/Geog461W.SP2014.Labs/tree/master/Demos/Animation) demo from last week, adding a PCP and customizing some of the labels using the [Lealfet **divIcon**()](https://www.mapbox.com/mapbox.js/example/v1.0.0/divicon/) function.
