# isvg2geojson

Converts an SVG file with added geo-referencing tags into a GeoJSON file. It filps the latitude coordiantes so that the GeoJSON output file can be loaded into Leaflet map.

The program is a small modification of [Phrognoz's svg2geojson program](https://github.com/Phrogz/svg2geojson).

## Installing

Clone this repository. See the [Cloning a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) documentation.

## Geo-Reference Tags
You still must still add `GeoItems` inside a Prognoz `MetaInfo` element as a direct child of the `<svg>` element.

~~~xml
<MetaInfo xmlns="http://www.prognoz.ru"><Geo>
  <GeoItem X="0" Y="0" Latitude="0" Longitude="0"/>
    <GeoItem X="2292" Y="2480" Latitude="2480" Longitude="2292"/>
</Geo></MetaInfo>
~~~

where `2292` is the image width and `2480` is the image height. See Phrognoz's documentation below. 

## Usage

At the top of the repository folder, run the binary in a bash terminal:

~~~
$ ./bin/isvg2geojson -i  file.svg             # Writes file.geojson
~~~

Note the preceeding 'i' in 'isvg2geojson' and the `-i` option.

All the options:

~~~
 -t, --tolerance=0.1 Distance (world meters) to sample curves to. (default: 0.1)
 -l, --layers        Split top-level groups into separate files. (default: one file)
 -o, --stdout        Output GeoJSON to stdout. (default: write files with names based on the SVG/layer)
 -m, --minimal       Make the GeoJSON as small as possible (not pretty).
 -p, --precision     Number of decimal places to format JSON numbers to. (default:6, as GeoJSON recommends)
 -d, --debug         Include ids for each SVG element in the features. (default: no properties)
 -v, --version       Output the version of svg2geojson as the first line on stdout (0.7.2).
 -h, --help          Show this help message.
 -i, --image         Flip the Y axis to use with Leaflet. (default: false)
~~~

When working with image overlay maps, I reccommend using high 'tolerance' and low 'precision', such as

~~~
$ ./bin/isvg2geojson -t=10000 -p=2 -i file.svg
~~~

This will make a reasonable sized output file with sufficient accurarcy to use in Leaflet image overlay maps. 

I also reccomend not minimizing the file using the 'minimal' option if you want to edit the GeoJSON file.

# Orignal SVG 2 GeoJSON

Converts an SVG file with added geo-referencing tags into one or more GeoJSON files.


## Installing

`npm install svg2geojson`

## Geo-Referencing Tags

You must place two `GeoItems` inside a [Prognoz MetaInfo](http://help.prognoz.com/8.0/en/mergedProjects/Specifications/svgmapspecification/structure/svgmap_structure.htm) element as a direct child of the `<svg>` element at the root of your document.

~~~xml
<MetaInfo xmlns="http://www.prognoz.ru"><Geo>
  <GeoItem X="-595.30" Y="-142.88" Latitude="37.375593" Longitude="-121.977795"/>
  <GeoItem X="1388.66" Y=" 622.34" Latitude="37.369930" Longitude="-121.959404"/>
</Geo></MetaInfo>
~~~

These map opposing X/Y corners in your SVG coordinate space to Longitude/Latitude coordinates on the world. _Note that the SVG coordinate space has Y increasing down (toward the south), while Latitude increases upwards (towards the north)._

## Usage

**Running the binary from the command line**:

~~~
$ npm install -g svg2geojson
$ svg2geojson file.svg          # Writes file.geojson
$ svg2geojson file.svg --layers # Writes file.geojson, file-layer1Name.geojson, …
# See svg2geojson --help for more parameters
~~~

**Running as a Node.js library**:

~~~ js
const { geoFromSVGFile, geoFromSVGXML } = require('svg2geojson.js');

// …reading from file on disk
geoFromSVGFile( 'my.svg', layers => {
	layers.forEach( layer => {
		let json = JSON.stringify(layer.geo); // Turn JS object into JSON string
		console.log(`Layer Named: "${layer.name}"`);
		console.log(json);
	});
}, {layers:true, tolerance:0.5} );

// …processing SVG code as a string
const svg = `<svg xmlns="http://www.w3.org/2000/svg"><!-- ... --></svg>`;
geoFromSVGXML( svg, layer => {
	let json = JSON.stringify(layer.geo); // Turn JS object into JSON string
	console.log(json);
} );
~~~

See the output of `svg2geojson --help` for the options you can pass to the functions, and their default values.


## Preparing Paths

SVG allows `<path>` elements with an arbitrary number of overlapping subpaths, with some of them being 'positive' space and some 'negative' space. In SVG these subpaths may be oriented clockwise or counter-clockwise, and added in any order.

GeoJSON only allows a `Polygon` to have a single 'positive' subpath (and an arbitrary number of additional 'hole' subpaths). To make it easier for the code to detect which subpath is the 'positive' subpath you must currently:

1. Have only one positive subpath per `<path>`.
2. Ensure that the positive subpath is the first subpath in a `<path>`.


## TODO (AKA Known Limitations)

* Support modes of projection unmapping
* Support non-rectangular, inverse bilinear unmappings
* Add more command-line options to control JSON formatting.
* Treat `<g>` as `MultiPolygon`, `GeometryCollection`, or `MultiLineString` as appropriate. Currently items within a group are flattened as individual `Feature` items in the GeoJSON.
* Treat `<path>` with multiple positive subpaths as a `MultiPolygon`. (This requires figuring out which holes apply to which positive subpaths.)
