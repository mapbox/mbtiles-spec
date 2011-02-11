# Formatter

The UTFGrid specification puts no constraints on the properties and type of data stored in values, only that the data is valid JSON. Formatter functions are used to provide application-specific representations of the data.

### Schema

Formatters are represented by a single row in the `metadata` table, with the `key=formatter` and `value=javascript function`.

### Data

Only a single formatter is permitted per tileset. The formatter function can be used for different presentations (hover, click, etc), by passing an `options` object as the first parameter to the function. The second parameter must be the JSON object from the dataset, in the form of an object.

     function (options, data) {
        ...
        return formatted_output;
     }

The `options` argument has one standardized property, `format`, which is expected to be a string, and all formatters must support two values, "teaser" and "full".

### Example

    function (options, data) {
        if (options.format == 'teaser') {
            return '<h1>' + data.NAME + '</h1>';
        } else if (options.format == 'full') {
            return '<h1>' + data.NAME + '</h1>' + data.AREA;
        }
    }
