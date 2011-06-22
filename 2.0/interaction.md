# Interaction

Tile servers can enhance tilesets with interactivity by implementing two additional HTTP endpoints.

* `[base path]/layer.json`: A layer manifest JSON containing the interaction formatter function and other optional attributes.
* `[base path]/{n}/{n}/{n}.grid.json`: A UTFGrid JSON file corresponding to its adjacent tile image.

`[base path]` refers to the full layer URL prior to any x, y or z coordinates.

Examples:

TMS-style URL schema

    http://example.com/1.0.0/world/0/0/0.png       // tile image for 0/0/0
    http://example.com/1.0.0/world/0/0/0.grid.json // utfgrid for 0/0/0
    http://example.com/1.0.0/world/layer.json      // layer manifest

OSM-style URL schema

    http://example.com/0/0/0.png       // tile image for 0/0/0
    http://example.com/0/0/0.grid.json // utfgrid for 0/0/0
    http://example.com/layer.json      // layer manifest

## layer.json

The layer manifest should provide a JSON object with the following keys:

* `formatter`: Array of strings. Anonymous javascript functions for formatting feature data from UTFGrid JSON. **Required**. 
* `legend`: Array of strings. Self-contained HTML that may be displayed as a legend for this layer. **Optional**.

Example response from `layer.json`:

    {
        "formatter": "function(options, data) { return data.NAME; }",
        "legend": "<strong>Countries of the World</strong>"
    }

Each `layer.json` item should be represented by a single row in the `metadata` table where `key,value` match its key and value in the `layer.json` object.

### Terminology

From this point on, **layer** is to refer to a datasource exposed by a **tileset**, such as those sourced from MBTiles files. **Tileset** is an HTTP resource that can exposed multiple layers in the same request by the UTFGrid 2.0 standard..

### formatter

The UTFGrid specification puts no constraints on the properties and type of data stored in values, only that the data is valid JSON. Formatter functions are used to provide application-specific representations of the data.

Only a single formatter is permitted per layer. The formatter function can be used for different presentations (hover, click, etc), by passing an `options` object as the first parameter to the function. The second parameter must be the JSON object from the dataset, in the form of an object.

     function (options, data) {
        ...
        return formatted_output;
     }

The `options` argument has one standardized property, `format`, which is expected to be a string, and all formatters must support two values, "teaser" and "full".

    function (options, data) {
        if (options.format == 'teaser') {
            return '<h1>' + data.NAME + '</h1>';
        } else if (options.format == 'full') {
            return '<h1>' + data.NAME + '</h1>' + data.AREA;
        }
    }

### legend

A tileset may provide an HTML string that can be rendered by the client as a legend. The string should be self-contained and not reference external stylesheets, scripts or images. The [Data URI scheme](http://en.wikipedia.org/wiki/Data_URI_scheme) may be used to embed images or other data if necessary.

    <div><span style='padding:0px 10px; background:#333;'></span> +10% population</div>
    <div><span style='padding:0px 10px; background:#666;'></span> +5% population</div>
    <div><span style='padding:0px 10px; background:#999;'></span> +0% population</div>
    <div><span style='padding:0px 10px; background:#ccc;'></span> -5% population</div>

## grid.json

See `utfgrid.md` for the format and storage of UTFGrid JSON.

