# Windy API (client v17+)

<!-- toc -->

- [About Windy API](#about-windy-api)
- [Module: $](#module-)
- [Module: map](#module-map)
  * [map.myMarkers: Predefined Leaflet markers](#mapmymarkers-predefined-leaflet-markers)
- [Class: Evented](#class-evented)
- [Module: brodacast](#module-brodacast)
  * [Main broadcasted messages are:](#main-broadcasted-messages-are)
- [Module: store](#module-store)
  * [Main items stored in store](#main-items-stored-in-store)
- [Module: overlays](#module-overlays)
  * [overlays.ident.convertMetric(number,separator)](#overlaysidentconvertmetricnumberseparator)
  * [overlays.ident.convertMetric(number)](#overlaysidentconvertmetricnumber)
- [Module: utils](#module-utils)
  * [utils.loadScript(url)](#utilsloadscripturl)
  * [utils.wind2obj](#utilswind2obj)
  * [utils.wave2obj](#utilswave2obj)
- [Module: plugins](#module-plugins)
- [Module: picker](#module-picker)
  * [Converting raw meteorological values to readable numbers](#converting-raw-meteorological-values-to-readable-numbers)

<!-- tocstop -->

## About Windy API
Windy codes consist of classes (for example `Evented`),  modules (for example `metrics`) and external plugins, that are loaded when necessary. The plugins are either internal (created by us) and external, created by Windy users.

Example of loading a module to your plugin:
```js
  import map from '@windy/map'
```

> While plugin codes use `import` keyword, Windy client uses its own
> dependency injection system. If you plan to compile plugin on
> your own, use `const map = W.require('map')`

## Module: $
Handy shortcut to `document.querySelector`.

Usage:
```js
  const el = $('.closing-x', parentEl ) // parentEl is optional
```

## Module: map
Instance of Leaflet map is available as `map` module. Windy uses Leaflet version `0.7.7` that is [well documented here](http://leafletjs.com/reference-0.7.7.html) and contains plenty of [plugins that you can use](http://leafletjs.com/plugins.html).

> We have tried to upgrade to Leaflet v1.0.0 and later on to v1.3.4, but
> we have found both version significantly slower, and containing major
> design flaws making it unusable on Windy. Also we miss some plugins,
> that were not ported.

`map` has also some custom Windy methods and props attached to itself.

### map.myMarkers: Predefined Leaflet markers
Windy defines small set of predefined markers (instances of `L.DivIcon`) that you could potentionaly use. Those are:

**map.myMarkers.icon**
Pulsating icon, pulsating for short time

**map.myMarkers.pulsatingIcon**
Pulsating icon forever

**map.myMarkers.myLocationIcon**
Blue icon of user's location

**map.myMarkers.other**
Rounded circle

Example:
```js
  L.marker([ 50, 14 ], {
    icon: map.myMarkers.pulsatingIcon
  }).addTo( map );
```

## Class: Evented
Some of the Windy components are descendants of `Evented` and emit messages.

Recieving and emmiting messages has usuall syntax and methods: `on, off, once, emit`. (You can use handy aliases `fire` or `trigger` to emit messages if you are used to).

Just remember, that broadcasts emitted by `map` are in fact Leaflet's brodcast, not Windy's ones.

## Module: brodacast
Major Windy's emitter (instance of `Evented`), used for most important events.

### Main broadcasted messages are:

**mapChanged**
After Leaflet map has been panned or zoomed.

**paramsChanged**
When user changes some paramters (overlay, level, date etc...). Do not not use this event to start any intensive action since Windy now must load and render all the data. We recommend to use `redrawFinished` instead.

**redrawFinished**
Triggered when Windy has succesfully loaded and rendered requested data. Use this for triggering your own tasks.

**metricChanged**
After some of the units (wind, temp, ...) has been changed.

**rqstOpen, rqstClose, closeAll**
Requests to load and open or close plug-ins (see later)

**pluginOpend, pluginClosed**
Lazy loaded plugin was sucessully loaded and opend/closed

**redrawLayers**
Forces various renderers to render layers, for example after reconfiguring  color gradient, or changing particle animation settings.

**uiChanged**
Whenever User Interface has been changed. Information for other UI components to recalculate their respective sizes and adapt themselfs to new layout.

Example:
```js
  broadcast.on('redrawFinished', params => {
    // Wow Windy has finished rendering.
  })
```

## Module: store
All major parametrs and settings are stored inside `store`. It is sophisticated key, value store that checks your input for validity and maintains integrity of all the parameters.

Use methods `get` to read value from store, `set` to change value in the store and `on` to observe change of the value. Some of the items respond to method `getAllowed` to return array of allowed values.

Method `set` returns `true` if provided value was valid and was actually changed.

Store is instance of `Evented`.

Example:
```js
  var overlay = store.get('overlay')
  // 'wind' ... actually used overlay

  var allowedOverlays = store.getAllowed('overlay')
  // ['wind', 'rain', ... ] ... list of allowed values

  store.set('overlay','rain')
  // true ... Metric was changed to rain

  store.on('overlay', ovr => {
  // Message will be emited only if value is valid and actually changed
  console.log('Wow, overlay has been chnaged to', ovr)
})
```

### Main items stored in store
Each stored item have some default value. The values, taht can be considered as users's own settings, are **read only.** It is strictlly **prohibited to change user's settings** without his action and permition.

Some of the major items you could be interested in are:

**overlay**
Color weather overlay. Use `store.getAllowed('overlay')` to get list of allowed values.

**level**
Level used for actualy displayed overlay or isolines. To get list of available levels for current combination of overlay and data provider type `store.get('availLevels')`

**availLevels**
List of levels, that are available for current combination of product and overlay.

**acTime**
Accumulated time. Use `store.getAllowed('acTime')` to get list of allowed values.

**timestamp**
Timestamp of actual weather moment. Use freely and without hesitation.

Example:
```js
  var fiveHours = 5 * 60 * 60 * 1000
  store.set('timestamp', Date.now() + fiveHours )
```

**path** - read only
UTC string containing time of actually rendered data that are available for current overlay and weather model.

**isolines** - read only
Isolines displayed over the map. Use `store.getAllowed('isoline')` to get list of allowed values.

**product**
Product is set of weather data, that have same resolution, boundaries, time range and so on. For simplification, you can think of product as a weather model. Use `store.getAllowed('product')` to get list of allowed values.

**particlesAnim** - read only
Informs if animation of wind/waves particles is running over the map.

**usedLang** - read only
ISO language code of if language used.

**hourFormat** - read only
Time format, returns `12h` or `24h`.

**numDirection** - read only
Display directions in Weather picker as number or as a string (for example NW). returns `true` or `false`..

## Module: overlays
Defines main parameters required to render specific weather overlay. Each defined overlay contains all used overlays together with their `colors`, settings for `legend` and `metrics`. Remember it is prohibited change the user's metric.

### overlays.ident.convertMetric(number,separator)
Converts value provided in default meteorological metric into user's selected
metric as a string. Separator is optional separator in between number and unit.

### overlays.ident.convertMetric(number)
Converts value provided in default meteorological metric into user's selected metric as a pure number.

Example:
```js
  overlays.wind.metric
  // 'kt' .. actually selected metric for wind overlay

  overlays.wind.listMetrics()
  // ['kt', 'bft', 'm/s', 'km/h', 'mph'] .. available metrics

  overlays.wind.convertNumber(45,' ')
  // '87 m/s'

  overlays.wind.convertValue(45)
  // 87

  broadcast.on('metricChanged', (overlay,newMetric) => {
    // Any changes of metric can be observed here
  })
```

## Module: utils
Handy utilities

### utils.loadScript(url)
Loads external JS file, returns Promise

Example:
```js
utils.loadScript('https://unpkg.com/d3@5.7.0/dist/d3.min.js')
     .then( initGraph )
```

### utils.wind2obj(obj)
Converts raw meterological values into wind Object.

Example:
```js
  utils.wind2obj( [3.8534493726842562, 7.977292512444887, 0] )
  // { dir: 210.4334, wind: 10.2 }
```

### utils.wave2obj(obj)
Converts raw meterological values into wave Object.

## Module: plugins
Windy can use plugins, that are loaded whenever necessary, making core codes small and fast. Plugin can be for example javascript library, or some user feature (like menu sliding from the right side). We recommend to access these plugins just by emitting messages `rqstOpen` and `rqstClose` on major `broadcast`. Some of the plugins require parameters for opening.

Only few plugins can be safely exposed in Windy API like: `distance`, `picker` or `settings`, `detail`.

Example:
```js
  broadcast.fire('rqstOpen','detail',{ lat: 50, lon: 14 })
  // Opens weather detail

  broadcast.fire('rqstOpen','detail')
  // Closes weather detail
```

## Module: picker
Weather picker can be opend programatically as any other plugin by emiting request message: `broadcast.fire('rqstOpen','picker',{ lat: 50, lon: 14 })`.

If the picker is opened outside visible map, it is closed afterwards, and also paning so the picker gets outside map, leads to close of the picker. Picker emits message about its own state.

Example:
```js
  broadcast.fire('rqstOpen','picker',{ lat: 50, lon: 14 })
  // Opens the picker

  broadcast.fire('rqstClose','picker')
  // Closes the picker
```

Picker emits messages `pickerOpened`, `pickerClosed` and `pickerMoved`, while picker moved provides **raw meteorological values** in the picker location.

Example:
```js
picker.on('pickerOpened', latLon => {
    // picker has been opened at latLon coords
})

picker.on('pickerMoved', latLon => {
    // picker was dragged by user to latLon coords

    let { lat, lon, values, overlay } = picker.getParams()
    // -> 50.4, 14.3, 'wind', [ U,V, ]

    let windObject = utils.wind2obj( values )
    // { overlay: 'wind', values: [ 0.4, 0.75, 0] }

})

picker.on('pickerClosed', () => {
    // picker was closed
})
```

### Converting raw meteorological values to readable numbers
Raw meteorological units returned frpm weather picker are usually described in respective documentation for `ECMWF`, `GFS` or other used forecast models.

Most popular overlays have these values:

**wind**
Array `[ U, V ]`, where U and V are wind vectors in `m/s`. To compute wind magnitude and direction use `W.utils.wind2obj`.

Example:
```js
    utils.wind2obj( [3.8534493726842562, 7.977292512444887, 0] )
    // { dir: 210.4334, wind: 10.2 }
```

**temperature, dewPoint**
Temperature in `K`

**rain, rainAccumulation**
Rain for duration of 3 or selected accumulation period in `mm`

**waves, swell**
Array `[ U, V, size]` where U and V are direction vectors and wave size is in `m`. Period in seconds is computed as `Math.sqrt( U * U + V * V )`. Use `W.utils.wave2obj` to compute respective values

Example:
```js
    utils.wave2obj( [4.36770334, -6.40998, 2.426323] )
    // { dir: 325.9878, size: 2.4, period: 8 }
```

To convert values to user's selected metrics use `convertNumber` or `convertValue` methods of respective `W.overlays` instance. While `converNumber` just recalculates value and return a number, `convertValue` adds name of a metric and returns a string.

Example:
```js
    var ovr = overlays[ store.get('overlay') ]
    // ovr now contains instance of actualy selected overlay

    ovr.convertNumber( 10 )
    // 19

    ovr.convertValue( 10 )
    // "19kt"
```
Note: Used `markdown-toc` CLI to generate TOC