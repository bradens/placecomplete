Placecomplete
=============

A jQuery and Select2 plugin for location autocomplete powered by the Google Maps API

## Summary

At Khan Academy we needed a way to let users:

1. Specify their location in their public profiles
2. Remove a previously specified location in an intuitive way

When used in this context, some existing location autocomplete plugins have subtle bugs and UI shortcomings (see the [Background](#background) section for more details):

- [Google Places Autocomplete widget](https://developers.google.com/maps/documentation/javascript/places#places_autocomplete)
- [Ubilabs Geocomplete plugin](http://ubilabs.github.io/geocomplete/).

This widget aims to be user-friendly to both the end-user and the developer. Please leave feedback in Github Issues!

## Dependencies

Make sure to load the following libraries (and associated stylesheets and assets) before using Placecomplete:

1. [jQuery](http://jquery.com/)
2. [Select2](http://ivaynberg.github.io/select2/)
3. [Google Maps Places API](https://developers.google.com/places/documentation/autocomplete)

The following code is sufficient for including the Google Maps API:

    <script type="text/javascript"
          src="https://maps.googleapis.com/maps/api/js?libraries=places&amp;sensor=false"></script>

## Usage

Clone the repo and add the following files to your project:

- `jquery.placecomplete.js`
- `jquery.placecomplete.css`
- `powered-by-google.png`

The `powered-by-google.png` image is needed for [proper attribution to Google for usage of their Places API](https://developers.google.com/places/policies). It must be located in the same directory as the css file.

For example:

    <html>
    <head>
        <script src="jquery.placecomplete.js"></script>
        <link rel="stylesheet" href="jquery.placecomplete.css">
    </head>
    <body>
        <input id="location-picker" type="text" />

        <script>
            $(function() {
                var $el = $("#location-picker");
                $el.placecomplete({});
            });
        </script>
    </body>
    </html>

## Configuration

### Simple configuration and default values

    $el.placecomplete({});

is equivalent to this:

    $el.placecomplete({
        placeholderText: "City, State, Country",
        requestParams: {
            types: ["(cities)"]
        }
    });

See the next section for more details on the format of `requestParams`.

### Different request parameters for the Google Places API call

The `requestParams` object takes the format of the [Google Maps API AutocompletionRequest object](https://developers.google.com/maps/documentation/javascript/reference#AutocompletionRequest).

Note that `input` will always be set by the plugin to the user's text input.

    $el.placecomplete({
        placeholderText: "Business name",
        requestParams: {
            types: ["establishment"]
        }
    );

### Select2 options

Because Placecomplete instantiates Select2 on your element, you can supply [Select2 options](http://ivaynberg.github.io/select2/#documentation) directly:

    $el.placecomplete({
        width: "element",
        placeholderText: "Enter your location here"
    });

## Events

Placecomplete triggers three events:

### `placecomplete:selected`
When the user selects an item from the autocomplete.

The `placeResult` object takes the format of the [Google Maps API PlaceResult object](https://developers.google.com/maps/documentation/javascript/reference#PlaceResult) with one small exception.

It has an additional property, `display_text` (arbitrarily chosen by me), which is set to the text that appeared in the autocomplete dropdown list. This is necessary because for some place results, the `formatted_address` field does not always match the text that was shown in the autocomplete. It appears to be a weird Google Places API quirk/bug.

    $el.on({
        "placecomplete:selected": function(evt, placeResult) {
            console.log(placeResult);
        }
    });

### `placecomplete:cleared`
When the user clears the input via the clear button

    $el.on({
        "placecomplete:cleared": function(evt) {
            console.log("Input cleared!");
        }
    });

### `placecomplete:error`
When the Google Maps API returns an error. The error message returned is a string equal to one of the [`google.maps.places.PlacesServiceStatus` constants](https://developers.google.com/maps/documentation/javascript/reference#PlacesServiceStatus).

    $el.on({
        "placecomplete:error": function(evt, errorMsg) {
            if (errorMsg ===
                google.maps.places.PlacesServiceStatus.INVALID_REQUEST) {
                // do something
            }
        }
    })

## Background

Unfortunately, existing plugins had bugs or UI shortcomings that made them less than ideal for our needs.

### [Google Places Autocomplete widget](https://developers.google.com/maps/documentation/javascript/places#places_autocomplete)

The [Google Places Autocomplete widget](https://developers.google.com/maps/documentation/javascript/places#places_autocomplete ([demo here](https://developers.google.com/maps/documentation/javascript/examples/places-autocomplete) had the following bugs and UI shortcomings:

1. Pressing enter in or deselecting (either by tabbing or clicking away from) the input field does not cause the first element to be selected.

2. There's no intuitive way to remove a previously specified location.

3. The PlaceResult object returned does not have a property that always matches the text originally shown to the user in the autocomplete. The property that comes closes to matching the autocomplete item text is `formatted_address`, which often but does not always match the autocomplete text. One example is the city of Sgurgola in Italy. It appears as "Sgurgola, Province of Frosinone, Italy" in the Google autocomplete, but the `formatted_address` property is set to "03010 Sgurgola Frosinone, Italy", and the original string is nowhere to be found in the PlaceResult object.

### [Ubilabs Geocomplete plugin](http://ubilabs.github.io/geocomplete/)

The [Ubilabs Geocomplete plugin](http://ubilabs.github.io/geocomplete/) uses the Google Places Autocomplete plugin and tries to fix problem #1, such that pressing enter selects the first item in the list. However, it was inadequate for our needs because of the following shortcomings:

1. Pressing enter in the input sometimes causes the wrong item to be selected because it makes a call to a different Google API than the one that powers the autocomplete results. For example, typing "chic" yields "Chicago, IL" as the first result, but pressing enter causes "Chic, 31310 Castagnac, France" to populate the input.

2. More significantly, when the text input is deselected (blurred) after using enter to select the first item, the input text returns to the input before the autocomplete due to some event handler registered by the Google Places Autocomplete widget.

3. Same problems as #2 and #3 for the Google Places Autocomplete widget.

## Approach

To avoid problems caused by the Google Places Autocomplete widget, we decided to just make the API calls to get the autocomplete results directly. Using Select2 for the interface component made the addition of a user-friendly "clear"/remove feature super simple (setting the `allowClear` property to `true`).

## Authors

[@stchangg](http://github.com/stchangg) - Developer at Khan Academy