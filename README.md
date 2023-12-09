# getPathData2

A collection of javaScript helpers for parsing and converting SVG path data from `SVGGeometryElement`s or `d` strings.

Based on the [W3C SVG working group's  `SVGPathData` interface` draft](https://svgwg.org/specs/paths/#InterfaceSVGPathData) 

Currently `getPathData()` and `setPathData()` are not natively supported by any major browser, thus you need a polyfill like [Jarek Foksa's path-data-polyfill](https://github.com/jarek-foksa/path-data-polyfill). 

This repository includes alternative standalone methods of the `getPathData()` and `setPathData()` but can also be used alongside the aforementioned polyfill.

This repository aims at providing helpers to:

* parse pathData from `d` attribute strings to a `SVGPathData` compliant array of command objects
* normalizing shorthand commands like `h`, `v`, `s`, `t` to longhanghand equivalents `l`, `c`, `q`
* convert quadratic to cubic commands
* convert `a` (arcto) commands to cubic Béziers
* convert commands to all absolute or relative commands
* round path data coordinates
* convert commands to shorthands if applicable
* convert primitives like `<circle>`, `<rect>`, `<polyline>` to `<path>` elements