# <img alt="benchpress" src="https://cdn.rawgit.com/benchpressjs/templates.js/benchpress/benchpress.svg" />

[![Build Status](https://travis-ci.org/benchpressjs/benchpress.png?branch=master)](https://travis-ci.org/benchpressjs/benchpress)
[![Code Climate](https://codeclimate.com/github/benchpressjs/benchpress.png)](https://codeclimate.com/github/benchpressjs/benchpress)

Benchpress is an ultralight (1.3kb minified) and super fast templating framework for Javascript and node.js.

It has [express](http://expressjs.com/) support out-of-the-box, and requires zero client-side dependencies.

## Installation
Benchpress is available as an npm module:

    npm i benchpressjs

## API
Benchpress uses an ahead of time (AOT) compilation model. It requires that you precompile templates into Javascript modules before using them.

### `.precompile({ source, minify = false }, callback)`
This method compiles a template source into Javascript code, optionally minifying the result with UglifyJS

```js
const benchpress = require('benchpressjs');
const template = 'My favourite forum software is {forum}. This templating engine is written in {language}.';
benchpress.precompile(template, (err, precompiled) => {
	// store it somewhere
});

// precompiled output
(function (factory) {
	if (typeof module === 'object' && module.exports) {
		module.exports = factory();
	} else if (typeof define === 'function' && define.amd) {
		define(factory);
	}
})(function () {
	function compiled(helpers, context, get, iter, helper) {
		return 'My favourite forum software is ' + get(context, ['forum']) + '. This templating engine is written in ' + get(context, ['language']) + '.';
	}

	return compiled;
});
```

### `.__express`

This method provides an express engine API.

```js
const express = require('express');
const app = express();
const benchpress = require('benchpressjs');

const data = {
	foo: 'bar',
};

app.configure(function() {
	app.engine('jst', benchpress.__express);
	app.set('view engine', 'jst');
	app.set('views', 'path/to/compiled/templates');
});

app.render('myview', data, function(err, html) {
	console.log(html);
});

app.get('/myroute', function(res, req, next) {
	res.render('myview', data);
});
```

### `.parse(template, data, callback)`

This method is used mainly to parse templates on the client-side.
To use it, `.registerLoader(loader)` must be used to set the callback for fetching compiled template modules.

```js
require(['benchpress'], (benchpress) => {
	benchpress.registerLoader((name, callback) => {
		// fetch `name` template module
	});

	benchpress.parse('basic', {
		forum: 'NodeBB',
		language: 'Javascript',
	}, (output) => {
		// do something with output
	});
});
```

This has been a quick rundown of the API. [See the full docs](docs.md)

## Template Syntax
Sample data, see test cases for more:

```json
{
	"animals": [
		{
			"name": "Cat",
			"species": "Felis silvestris catus",
			"isHuman": false,
		},
		{
			"name": "Dog",
			"species": "Canis lupus familiaris",
			"isHuman": false,
		},
		{
			"name": "Human",
			"species": "Homo sapiens",
			"isHuman": true
		}
	],
	"package": {
		"name": "benchpressjs",
		"author": "psychobunny",
		"url": "http://www.github.com/benchpressjs/benchpress"
	},
	"website": "http://burnaftercompiling.com",
	"sayHello": true
}
```

### Simple key/value
```
My blog URL is {website}. The URL for this library is {{package.url}}
```
Double brackets and single brackets are identical since there is no longer a use case for escaping **templates.js** tokens.

### Conditionals
```html
<!-- IF sayHello -->
  Hello world!
<!-- END -->

<!-- IF !somethingFalse -->
  somethingFalse doesn't exist
<!-- END -->
```
Benchpress supports several syntaxes for conditionals in order to be backwards compatible with **templates.js**.
`<!-- ENDIF abcd -->`, `<!-- END abcd -->`, `<!-- ENDIF !foobar -->`, and `<!-- END -->` are all equivalent tokens as far as Benchpress is concerned.

### Iteration
Repeat blocks of HTML. The two special keys `@first` and `@last` are available as booleans, and the `@index`, `@key`, and `@value` special keys are also available. Benchpress supports iterating over objects, in which case `@index` will be the current loop number and `@key` will be the key of the current item. For normal arrays, `@key == @index`.

```html
<!-- BEGIN animals -->
  {animals.name} is from the species {animals.species}.
  <!-- IF !animals.isHuman -->
    - This could be a pet.
  <!-- ENDIF !animals.isHuman -->
<!-- END animals -->

prints out:

Cat is from the species Felis silvestris catus.
- This could be a pet.
Dog is from the Canis lupus familiaris.
- This could be a pet.
Human is from the species Homo sapiens.
```

Benchpress supports several syntaxes for iteration in order to be backwards compatible with **tempates.js**:
 - `<!-- END abcd -->` == `<!-- END foo -->` == `<!-- END -->`
 - `<!-- BEGIN abc --> {abc.def} <!-- END -->` == `<!-- BEGIN abc --> {../def} <!-- END -->` which will print the `def` key of every item in `abc`.

There is a grey zone where if you wish to print a field of the object you are iterating over, you can't directly. This is a breaking change from **templates.js**. To fix this, change `{abc.def}` to `{../../abc.def}`.

### Helpers

Helpers are JavaScript methods for advanced logic in templates. This example shows a really simple example of a function called `print_is_human` which will render text depending on the current block's data.

```js
benchpress.registerHelper('print_is_human', function (data) {
  return (data.isHuman) ? "Is human" : "Isn't human";
});
```

```html
<!-- BEGIN animals -->
{function.print_is_human}
<!-- END animals -->

prints out:

Isn't human
Isn't human
Is human
```

## Testing

    npm install
    npm test

## Projects using Benchpress

[NodeBB](http://www.nodebb.org)

Add yours here by submitting a PR :)
