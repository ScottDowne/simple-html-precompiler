# simple-html-precompiler
webpack plugin to precompile html

# Plain webpack.config.js example
```
var SimpleHtmlPrecompiler = require('simple-html-precompiler');
var path = require('path');
module.exports = {
  entry: "./client.jsx",
  output: {
    filename: '[name].js',
    chunkFilename: '[id].chunk.js',
    path: path.join('public')
  },
  module: {
    loaders: [
      { test: /\.jsx$/, loaders: ['jsx-loader'] },
      { test: /\.json$/, loaders: ['json-loader'] }
    ]
  },
  plugins: [
    new SimpleHtmlPrecompiler(['/', 'hello-world'], function(outputPath, callback) {
      callback("<div>" outputPath "</div>");
    })
  ]
};
```

# API usage
`SimpleHtmlPrecompiler(path, process)`

* `path` - An array of string for all the paths you want to compile
* `process` - A function called for each item in the path. Params are `outputPath` and `callback`
* `outputPath` - A string for the current path.
* `callback` - Usage `callback(html)` call this with the HTML string you want compiled.

# With React
webpack.config.js
```
var SimpleHtmlPrecompiler = require('simple-html-precompiler');
var path = require('path');
var React = require('react');
require('node-jsx');
var Router = require('react-router');
var routes = require('./routes.jsx');
module.exports = {
  entry: "./client.jsx",
  output: {
    filename: '[name].js',
    chunkFilename: '[id].chunk.js',
    path: path.join('public')
  },
  module: {
    loaders: [
      { test: /\.jsx$/, loaders: ['jsx-loader'] },
      { test: /\.json$/, loaders: ['json-loader'] }
    ]
  },
  plugins: [
    new SimpleHtmlPrecompiler(['/', 'hello-world'], function(outputPath, callback) {
      Router.run(routes, outputPath, function (Handler, state) {
        var Index = React.createFactory(require('./pages/index.jsx'));
        var Page = React.createFactory(Handler);
        callback(React.renderToString(Index({
          markup: React.renderToString(Page())
        })));
      });
    })
  ]
};
```
pages/index.jsx
```
var React = require('react');

var Index = React.createClass({
  render: function() {
    return (
      <html>
        <head>
          <meta charSet="UTF-8"/>
          <meta name="viewport" content="width=device-width, initial-scale=1"/>
          <title>Title</title>
          <link rel="stylesheet" href="/css/index.css"/>
        </head>
        <body>
          <div id="my-app" dangerouslySetInnerHTML={{__html: this.props.markup}}></div>
          <script src="/main.js"></script>
        </body>
      </html>
    );
  }
});

module.exports = Index;
```
routes.jsx
```
var React = require('react');
var Route = require('react-router').Route;

var routes = (
  <Route>
    <Route name="home" path="/?" handler={require('./pages/home.jsx')} />
    <Route name="hello-world" path="/hello-world/?" handler={require('./pages/hello-world.jsx')} />
  </Route>
);

module.exports = routes;
```
client.jsx
```
var React = require('react'),
    Router = require('react-router');

var routes = require("./routes.jsx");

Router.run(routes, Router.HistoryLocation, function (Handler, state) {
  React.render(<Handler/>, document.querySelector("#my-app"));
});
```
