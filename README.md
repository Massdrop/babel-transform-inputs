# babel-plugin-transform-imports

NOTE: This is a fork by Massdrop of https://bitbucket.org/amctheatres/babel-transform-imports

It is only published to an internal NPM registry, in order to use the changes from this PR: https://bitbucket.org/amctheatres/babel-transform-imports/pull-requests/5

Transforms member style imports:

```javascript
    import { Row, Grid as MyGrid } from 'react-bootstrap';
    import { merge } from 'lodash';
```

...into default style imports:

```javascript
    import Row from 'react-bootstrap/lib/Row';
    import MyGrid from 'react-bootstrap/lib/Grid';
    import merge from 'lodash/merge';
```

*Note: this plugin is not restricted to the react-bootstrap and lodash
libraries.  You may use it with any library.*

## That's stupid, why would you do that?

When Babel encounters a member style import such as:

```javascript
    import { Grid, Row, Col } from 'react-bootstrap';
```

it will generate something similarish to:

```javascript
    var reactBootstrap = require('react-bootstrap');
    var Grid = reactBootstrap.Grid;
    var Row = reactBootstrap.Row;
    var Col = reactBootstrap.Col;
```

Some libraries, such as react-bootstrap and lodash, are rather large and
pulling in the entire module just to use a few pieces would cause unnecessary
bloat to your client optimized (webpack etc.) bundle.  The only way around
this is to use default style imports:

```javascript
    import Grid from 'react-bootstrap/lib/Grid';
    import Row from 'react-bootstrap/lib/Row';
    import Col from 'react-bootstrap/lib/Col';
```

But, the more pieces we need, the more this sucks.  This plugin will allow you
to pull in just the pieces you need, without a separate import for each item.
Additionally, it can be configured to throw when somebody accidentally writes
an import which would cause the entire module to resolve, such as:

```javascript
    import Bootstrap, { Grid } from 'react-bootstrap';
    // -- or --
    import * as Bootstrap from 'react-bootstrap';
```

## Installation

```
npm install --save-dev babel-plugin-transform-imports
```

## Usage

*In .babelrc:*

```json
    {
        "plugins": [
            ["transform-imports", {
                "react-bootstrap": {
                    "transform": "react-bootstrap/lib/${member}",
                    "preventFullImport": true
                },
                "lodash": {
                    "transform": "lodash/${member}",
                    "preventFullImport": true
                }
            }]
        ]
    }
```

## Advanced Transformations

In cases where the provided default string replacement transformation is not
sufficient (for example, needing to execute a RegExp on the import name), you
may instead provide a path to a .js file which exports a function to run
instead.  Keep in mind that the .js file will be `require`d relative from this
plugin's path, likely located in `/node_modules/babel-plugin-transform-imports`.
You may provide any filename, as long as it ends with `.js`.

.babelrc:
```json
    {
        "plugins": [
            ["transform-imports", {
                "my-library": {
                    "transform": "../../path/to/transform.js",
                    "preventFullImport": true
                }
            }]
        ]
    }
```

/path/to/transform.js:
```js
module.exports = function(importName) {
    return 'my-library/etc/' + importName.toUpperCase();
};
```

This is a little bit hacky, but options are a bit limited due to .babelrc being
a JSON5 file which does not support functions as a type.  In Babel 7.0, it
appears .babelrc.js files will be supported, at which point this plugin will be
updated to allow transform functions directly in the configuration file.
See: https://github.com/babel/babel/pull/4892

## Webpack

This can be used as a plugin with babel-loader.

webpack.config.js:
```js
module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
            loader: 'babel-loader',
                query: {
                    plugins: [
                        [require('babel-plugin-transform-imports'), {
                            "my-library": {
                                "transform": function(importName) {
                                    return 'my-library/etc/' + importName.toUpperCase();
                                },
                                preventFullImport: true
                            }
                        }]
                    ]
                }
            }
        }
    ]
}
```

## Options

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `transform` | `string` | yes | `undefined` | The library name to use instead of the one specified in the import statement.  ${member} will be replaced with the member, aka Grid/Row/Col/etc.  Alternatively, pass a path to a .js file which exports a function to process the transform (see Advanced Transformations) |
| `preventFullImport` | `boolean` | no | `false` | Whether or not to throw when an import is encountered which would cause the entire module to be imported. |
| `camelCase` | `boolean` | no | `false` | When set to true, runs ${member} through _.camelCase. |
| `kebabCase` | `boolean` | no | `false` | When set to true, runs ${member} through _.kebabCase. |
| `snakeCase` | `boolean` | no | `false` | When set to true, runs ${member} through _.snakeCase. |
| `skipDefaultConversion` | `boolean` | no | `false` | When set to true, will preserve `import { X }` syntax instead of converting to `import X`. |
