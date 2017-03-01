# Webpack 2 Tutorial

Following tutorial https://blog.madewithenvy.com/getting-started-with-webpack-2-ed2b86c68783#.qux99et2b

Webpack tries to make the build process easier by passing dependencies through JavaScript.

# Create package.json

```
$ npm init
```

Install webpack and webpack-dev-server:

```
$ npm i webpack webpack-dev-server --save-dev
```

Create a .gitignore file:
[.gitignore](.gitignore)

Create a webpack.config.js file:
[webpack.config.js](webpack.config.js)

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: path.resolve(__dirname, './src'),
  entry: {
    app: './app.js',
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name].bundle.js',
  },
};
```

`__dirname` refers to the directory where the `webpack.config.js` file is.

## What webpack does?

1. Starts from `context` folder,
2. Looks for `entry` filenames and its contents,
3. It looks for every `import` or `require()` dependency and parses the code to bundle it for the final build.
4. After parsing all files, webpack bundles everything to `output.path` folder, naming it using the `output.filename` naming template (`[name]` gets replaced with the object key from `entry`)

If you have a `src/app.js` file, you now can run:

```
$ webpack -p
```

The `p` flag is production mode and uglifies the output.

## Working with multiple files

### Multiple files, bundled together

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: path.resolve(__dirname, './src'),
  entry: {
    app: ['./home.js', './events.js', './vendor.js'],
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name].bundle.js',
  },
};
```

All will be bundled together into `dist/app.bundle.js`.

### Multiple files, multiple outputs

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: path.resolve(__dirname, './src'),
  entry: {
    home: './home.js',
    events: './events.js',
    contact: './contact.js',
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name].bundle.js',
  },
};
```

This will be bundled as 3 files: `dist/home.bundle.js`, `dist/events.bundle.js`, and `dist/contact.bundle.js`.

#### Automatic vendor bundling

Sometimes when you create separated bundles, you might have repeated code among those files, usually vendor libraries.
Webpack let you make a separated bundle for the vendor code with CommonsChunk plugin.

```javascript
module.exports = {
  //...
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'commons',
      filename: 'commons.js',
      minChunks: 2,
    }),
  ],
  //...
}
```

Now, across your `output` files, if you have any modules that get loaded 2 or more times, it will bundle that into a `commons.js` file.

#### Manual vendor bundling

You can also take a more manual approach:

```javascript
module.exports = {
  entry: {
    index: './index.js',
    vendor: ['react', 'react-dom', 'rxjs'],
  },
}
```

In this, you're explicitly telling webpack to export a `vendor` bundle containing your `react` related node modules, rather than relying on CommonsChunkPlugin to do its thing automatically.

## Development

To get the development server running, just add a `devServer` object to `webpack.config.js`:

```javascript
module.exports = {
  context: path.resolve(__dirname, './src'),
  entry: {
    app: './app.js',
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, './dist/assets'),
    publicPath: '/assets',                          // New
  },
  devServer: {
    contentBase: path.resolve(__dirname, './src'),  // New
  },
};
```

Now make a `src/index.html` file with:

```html
<script src="/assets/app.bundle.js"></script>
```

And run the webpack server:

```
$ ./node_modules/.bin/webpack-dev-server
```

Your server is now running at `localhost:8080`.

## Globally-accessible methods
