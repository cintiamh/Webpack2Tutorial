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

`__dirname` refers to the directory where the `webpack.config.js` file is.

## What webpack does?

1. Starts from `context` folder,
2. Looks for `entry` filenames and its contents,
3. It looks for every `import` or `require()` dependency and parses the code to bundle it for the final build.
4. After parsing all files, webpack bundles everything to `output.path` folder, naming it using the `output.filename` naming template (`[name]` gets replaced with the object key from `entry`)
