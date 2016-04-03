# Webpack Isomorphic Loader

Webpack loader and tools to extend NodeJS `require` so it understands your web asset files such as images when you are doing server side rendering.


## Install

```
$ npm install isomorphic-loader --save
```

## Purpose

With [webpack] and [file-loader], you can do things like this in your React code:

```js
import smiley from "./images/smiley.jpg";


render() {
    return <div><img src={smiley} /></div>
}
```

That works out nicely, but if you need to do Server Side Rendering, you will get SyntaxError from Node `require`.  That's because `require` only understand JS files.

With this module, you can extend Node's `require` so it will understand these files.

It saves a list of your files that you want to be treated as assets and the extended `require` will return the URL string like it would on the client side.

## Usage

### Configuring Webpack

First you need to mark all your asset files that you want `extendRequire` to handle.  To do that, simply use the webpack loader `isomorphic-loader` on the files.

> The webpack loader `isomorphic-loader` is just a simple pass thru loader to mark your files.

You also need to install a plugin to collect and save the list of the files you marked.

For example, in the webpack config, to mark the usual image files to be understood by the `extendRequire`:

```js
var IsomorphicLoaderPlugin = require("isomorphic-loader/lib/webpack-plugin");

module.exports = {
    plugins: [
        new IsomorphicLoaderPlugin()
    ],
    module: {
        loaders: [
            {
                test: /\.(jpe?g|png|gif|svg)$/i,
                loader: "file!isomorphic"
            }
        ]
    }
};
```

You can also mark any file in your code directly:

```js
import smiley from "file!isomorphic!./images/smiley.jpg";
```

After webpack compiled your project, the plugin will generate a config file `.isomorphic-loader-config.json` in your `CWD` and a JSON file with mappings of the asset files you marked.

See [more details about config and assets mapping files](#config-and-assets-files).


### Extending Node Require

To use the asset files mapping for SSR with Node, you need to extend `require` before your server starts.

To do that:

```js
var extendRequire = require("isomorphic-loader/lib/extend-require");

extendRequire(function (err) {
    if (err) {
        console.log(err);
    } else {
        require("./server");
    }
});
```

If `Promise` is supported:

```js
extendRequire().then(function () {
    require("./server");
}).catch(function (err) {
    console.log(err);
});
```

It will use the config file `.isomorphic-loader-config.json` in your `CWD` generated by the webpack plugin.

If the config file is not found, it will wait until it's generated.  This is so you can use [webpack-dev-server].

#### Reloading Assets for Extend Require

If at any time you wish `extendRequire` to reload the assets, you can use the `loadAssets` API.

```js
var extendRequire = require("isomorphic-loader/lib/extend-require");

extendRequire.loadAssets(true, function (err) {
    if (err) {
        console.log(err);
    }
});
```

### webpack-dev-server

[webpack-dev-server] is automatically detected and supported.

The webpack plugin accepts two options for [webpack-dev-server].

  * The URL to the server.  This is default to `http://localhost:8080`.

### Config and Assets Files

The webpack plugin creates a config file in your `CWD` and an assets mapping file in your webpack output directory.

The default name of the mapping file is `isomorphic-assets.json`.

You can configure the name when you initialize the plugin.  For example:

```js
new IsomorphicLoaderPlugin({assetsFile: "assets/isomorphic-assets.json"});
```

It will also save in the config the `publicPath` from your webpack config.

Here is how the generated config file might look like:

```json
{
  "version": "0.1.0",
  "context": "client",
  "debug": false,
  "devtool": false,
  "output": {
    "path": "dist",
    "filename": "bundle.js",
    "publicPath": "/test/"
  },
  "assetsFile": "dist/isomorphic-assets.json"
}
```


## License

[MIT License]


[file-loader]: https://github.com/webpack/file-loader
[MIT License]: http://www.opensource.org/licenses/mit-license.php
[webpack-dev-server]: https://webpack.github.io/docs/webpack-dev-server.html
[webpack]: https://webpack.github.io/

