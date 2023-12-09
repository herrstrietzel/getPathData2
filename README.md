# getPathData2

A collection of javaScript helpers for parsing and converting SVG path data from `SVGGeometryElement`s or `d` strings.

Based on the [W3C SVG working group's  `SVGPathData` interface` draft](https://svgwg.org/specs/paths/#InterfaceSVGPathData) 

Currently `getPathData()` and `setPathData()` are not natively supported by any major browser, thus you need a polyfill like [Jarek Foksa's path-data-polyfill](https://github.com/jarek-foksa/path-data-polyfill). 

This repository includes alternative standalone methods of the `getPathData()` and `setPathData()` but can also be used alongside the above polyfill.

This repository aims at providing helpers to:

* parse pathData from `d` attribute strings to a `SVGPathData` compliant array of command objects
* normalizing shorthand commands like `h`, `v`, `s`, `t` to longhanghand equivalents `l`, `c`, `q`
* convert quadratic to cubic commands
* convert `a` (arcto) commands to cubic Béziers
* convert commands to all absolute or relative commands
* round path data coordinates
* convert commands to shorthands if applicable
* convert primitives like `<circle>`, `<rect>`, `<polyline>` to `<path>` elements

## Usage
Load getPathData2 locally or via cdn

``` 
<script src="getPathData2.js"></script>
```

``` 
<script src="https://unpkg.com/getpathdatatwo@latest/getPathData2.js"></script>
```

### Parse
You can retrieve pathData from a `<path>` as well as other SVGGeometryElements such as `<rect>`, `<circle>`, `<ellipse>`, `<line>`, `<polyline>`, `<polygon>`. See also section ["Get path data from shapes/primitives"](#user-content-get-path-data-from-shapesprimitives)


```
let pathData = path.getPathData2();
```

`parseDtoPathData()` allows you to parse from any valid stringified pathdata

```
let d = ` m 50 0 .00001.0001.001 10e-10 Q 36.4 0 24.8 6.8 t -18 18 t -6.8 25.2 C 0 63.8 5.6 76.3 14.65 85.35 s 21.55 14.65 35.35 14.65 A 50 25 -45 0 1 100 40 h -20-.5 v -12.5-12.5 H 75 50V 0z`;

let pathDataD = parseDtoPathData(d);
```

### Normalize and convert
Similar to the official `getPathData()` method we can add a normalize parameter like so:  

```
let options = {normalize:true}
let pathDataNormalized = path.getPathData2(options)
```

This will apply the following conversions
* relative to absolute 
* shorthand to longhand/unshortened commands like `h`, `v`, `t`, `s`
* quadratic to cubic bézier
* `A` arctos to cubic bézier approximations

You can define more finegrained conversions by these parameters

| Parameter | Values/defaults | Effect | 
| --- | --- | --- |
| toAbsolute | Boolean; false | all commands to absolute |
| toRelative | Boolean; false | all commands to relative |
| arcToCubic | Boolean; false | all arcs to cubic |
| arcAccuracy | Number; 1 | add segments for better cubic approximation |
| quadraticToCubic | Boolean; false | quadratic commands to cubic |
| toLonghands | Boolean; false | shorthands to longhands |
| normalize  | Boolean; false | shorthand for: toAbsolute, arcToCubic, quadraticToCubic, toLonghands |

#### Example1: simple normalization
Quite often you need to all absolute values as well as longhand commands. By specifying only required conversions you retain more of the original path information than using the "brute-force" normalize option.  
Skipping the rather expensive arctocubic conversion also improves performance. 
(See examples/convert.html)

```
let options = {
    toAbsolute: true,
    toLonghands: true
}
let pathData = path.getPathData2(options);
```

#### Example2: individual conversion helpers
When parsing from strings you can combine all conversion helpers individually.  
(See examples/convert.html)

``` 
let d = ` m 50 0 .00001.0001.001 10e-10 Q 36.4 0 24.8 6.8 t -18 18         
t -6.8 25.2 C 0 63.8 5.6 76.3 14.65 85.35 s 21.55 14.65 35.35 14.65 A 50 25 -45 0 1 100 40 h -20-.5 v -12.5-12.5 H 75 50V 0z`;

let pathDataD = parseDtoPathData(d)

// to absolute
pathDataD = pathDataToAbsolute(pathDataD)

// to relative
pathDataD = pathDataToRelative(pathDataD)

// to longhands
pathDataD = pathDataToLonghands(pathDataD)

// to shorthands
pathDataD = pathDataToShorthands(pathDataD)

```

## Set/apply pathData

`setPathData2(options)` applies the stringified pathdata to a `<path>` element. Like the `getPathData2()` method you can specify options to get an optimized `d` attribute value.  
This is handy if you intend to minify the final pathdata output after previous manipulations (e.g. scaling, shifting).


| Parameter | Values/defaults | Effect | 
| --- | --- | --- |
| toAbsolute | Boolean; false | all commands to absolute |
| toRelative | Boolean; false | all commands to relative |
| arcToCubic | Boolean; false | all arcs to cubic |
| arcAccuracy | Number; 1 | add segments for better cubic approximation |
| quadraticToCubic | Boolean; false | quadratic commands to cubic |
| toLonghands | Boolean; false | shorthands to longhands |
| minify | Boolean; false | minify d string by removing whitespace omitting command tokens for repeated commands etc. |
| decimals | number; -1 | round to decimals. -1 == no rounding |
| optimize  | Boolean; false | shorthand for: toRelative, toShorthandshands, decimals=3 |


## Get path data from shapes/primitives
You can retrieve pathData from any `SVGGeometryElement`. If you need to replace a shape with a `<path>` element you can use the `convertShapeToPath()` method. (See examples/shapes.html). 
All attributes except those specific to shapes (e.g. `x`, `cx`, `width` etc.) are copied to the new `<path>` element.

```
// parse pathData
let options = {
    toRelative: true,
    toShorthands: true,
    arcToCubic: true,
    decimals: 1,
    minify: true
}

shape.convertShapeToPath(options)
```

### Credits

* Jarek Foksa for his [great polyfill](https://github.com/jarek-foksa/path-data-polyfill) heavily inspring to adopt the new pathData interface methodology and for contributing to the specification
* Dmitry Baranovskiy for (raphael.j/snap.svg) [pathToAbsolute/Relative functions](https://github.com/DmitryBaranovskiy/raphael/blob/master/raphael.js#L1848) 
* Puzrin (fontello) for the arc to cubic conversion method  [a2c.js](https://github.com/fontello/svgpath/blob/master/lib/a2c.js)
* Mike "POMAX" Kammermans for his great [A Primer on Bézier Curves](https://pomax.github.io/bezierinfo)
