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

If you want to access a function from a global namespace, simply set `output.library` within `webpack.config.js` and it will attach your bundle to a `window.myClassName` instance.

```javascript
module.exports = {
  output: {
    library: 'myClassName',
  }
};
```

## Loaders

We can work with virtually any file type, as long as we pass it into JavaScript with Loaders.

On NPM they are usually named `*-loader` such as `sass-loader` or `babel-loader`.

### Babel + ES6

First install:

```
$ npm i babel-loader babel-core babel-preset-es2015 --save-dev
```

Now include them into webpack.config.js file:

```javascript
module.exports = {
  // …
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: [/node_modules/],
        use: [{
          loader: 'babel-loader',
          options: { presets: ['es2015'] }
        }],
      },
    ],
  },
  // …
};
```

Webpack relies on regex tests to give you complete control.

You can use `include` as well to make the opposite exception.

### CSS + Style Loader

We can import the css from the js file:

```javascript
import styles from './assets/stylesheets/application.css'
```

We'll get the error:
`You may need an appropriate loader to handle this file type`.

Now we have to install and setup a css loader:

```
$ npm i css-loader style-loader --save-dev
```

And the webpack.config.js:

```javascript
module.exports = {
  // …
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```

Note that the loaders are processed in reverse array order. So `css-loader` will run before `style-loader`.

`css-loader` bundles the css into your javascript and `style-loader` manually writes your styles to the `<head>`.

### CSS + Node modules

```
@import "~normalize.css";
```

### CSS Modules

CSS Modules scopes your CSS classes to the JavaScript file that loaded it.
You'll need `css-loader` for this.

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: { modules: true }
          },
        ],
      },
    ],
  },
};
```

You can actually drop the `~` when importing Node Modules with CSS Modules enabled.

If you are getting `Can't find ...` errors, try adding a `resolve` to your webpack.config.js:

```JavaScript
module.exports = {
  resolve: {
    modules: [path.resolve(__dirname, './src'), 'node_modules']
  },
};
```

### Sass

```
$ npm i --save-dev sass-loader node-sass
```

webpack.config.js

```JavaScript
module.exports = {
  // …
  module: {
    rules: [
      {
        test: /\.(sass|scss)$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader',
        ]
      }
      // …
    ],
  },
};
```

### CSS bundled separately

install plugin:

```
$ npm install --save-dev extract-text-webpack-plugin
```

And in your webpack.config.js:

```javascript
const ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: "css-loader"
        })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin("styles.css"),
  ]
}
```

### HTML

As you can expect, there is a html-loader: https://github.com/webpack-contrib/html-loader
