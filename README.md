
# `iterare`

> lat. _to repeat, to iterate_

[![Version](https://img.shields.io/npm/v/iterare.svg)](https://www.npmjs.com/package/iterare)
[![Downloads](https://img.shields.io/npm/dt/iterare.svg)](https://www.npmjs.com/package/iterare)
[![Build Status](https://travis-ci.org/felixfbecker/iterare.svg?branch=master)](https://travis-ci.org/felixfbecker/iterare)
[![Coverage](https://codecov.io/gh/felixfbecker/iterare/branch/master/graph/badge.svg?token=BuoxrgBs54)](https://codecov.io/gh/felixfbecker/iterare)
[![Dependency Status](https://gemnasium.com/badges/github.com/felixfbecker/iterare.svg)](https://gemnasium.com/github.com/felixfbecker/iterare)
![Node Version](http://img.shields.io/node/v/iterare.svg)
[![License](https://img.shields.io/npm/l/iterare.svg)](https://github.com/felixfbecker/iterare/blob/master/LICENSE.txt)
[![Gitter](https://badges.gitter.im/felixfbecker/iterare.svg)](https://gitter.im/felixfbecker/iterare?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

ES6 Iterator library for applying multiple transformations to a collection in a single iteration.

## [API Documentation](http://iterare.surge.sh/)

## Motivation

Ever wanted to iterate over ES6 collections like `Map` or `Set` with `Array`-built-ins like `map()`, `filter()`, `reduce()`?
Lets say you have a large `Set` of URIs and want to get a `Set` back that contains file paths from all `file://` URIs.

The loop solution is very clumsy and not very functional:
```javascript
const uris = new Set([
  'file:///foo.txt',
  'http:///npmjs.com',
  'file:///bar/baz.txt'
])
const paths = new Set()
for (const uri of uris) {
  if (!uri.startsWith('file://')) {
    continue
  }
  const path = uri.substr('file:///'.length)
  paths.add(path)
}
```

Much more readable is converting the `Set` to an array, using its methods and then converting back:

```javascript
new Set(
  Array.from(uris)
    .filter(uri => uri.startsWith('file://'))
    .map(uri => uri.substr('file:///'.length))
)
```

But there is a problem: Instead of iterating once, you iterate 4 times (one time for converting, one time for filtering, one time for mapping, one time for converting back).
For a large Set with thousands of elements, this has significant overhead.

Other libraries like RxJS or plain NodeJS streams would support these kinds of "pipelines" without multiple iterations, but they work only asynchronously.

With this library you can use many methods you know and love from `Array` and lodash while only iterating once - thanks to the ES6 [iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols):

```javascript
import iterate from 'iterare'

iterate(uris)
  .filter(uri => uri.startsWith('file://'))
  .map(uri => uri.substr('file:///'.length))
  .toSet()
```

`iterate` accepts any kind of Iterator or Iterable (arrays, collections, generators, ...) and returns a new Iterator object that can be passed to any Iterable-accepting function (collection constructors, `Array.from()`, `for of`, ...).
Only when you call a method like `toSet()`, `reduce()` or pass it to a `for of` loop will each value get pulled through the pipeline, and only once.

This library is essentially
 - RxJS, but fully synchronous
 - lodash, but with first-class support for ES6 collections.

## Performance

[Benchmark](https://github.com/felixfbecker/iterare/blob/master/src/benchmark.ts) based on the example above:

Method                       | ops/sec
-----------------------------|-----------------------------------------------:|
Loop                         | 227 ops/sec ±2.30% (73 runs sampled)
**iterare**                  | **208 ops/sec ±3.53% (72 runs sampled)**
Array method chain           | 129 ops/sec ±1.25% (69 runs sampled)
Lodash (with lazy evalution) | 168 ops/sec ±1.50% (73 runs sampled)
RxJS                         | 152 ops/sec ±1.93% (76 runs sampled)

## Contributing

The source is written in TypeScript.

 - `npm run build` compiles TS
 - `npm run watch` compiles on file changes
 - `npm test` runs tests
 - `npm run benchmark` runs the benchmark
