This repo is a tutorial about how to build a react starter kit step by step.

All starter kits in this tutorial use ES6 by default.

Table of Contents
-----------------
1. [Kit1: React+Babel+Webpack](#1-kit1-reactbabelwebpack)
1. [Kit2: Kit1 + webpack-dev-server + ESLint](#2-kit2-kit1--webpack-dev-server--eslint)
1. [Kit3: Kit2 + Redux](#3-kit3-kit2--redux)
1. [Kit4: Kit3 + react-router](#4-kit4-kit3--react-router)
1. [Kit5: Kit4 + Mocha](#5-kit5-kit4--mocha)
1. [Kit6: Kit5 + material-ui](#6-kit6-kit5--material-ui)
1. [Kit7: Kit6 + redux-devtools](#7-kit7-kit6--redux-devtools)


# 1 Kit1: React+Babel+Webpack

First, make an empty project by running `npm init`, and enter the following information:

    mkdir kit1 && cd kit1
    npm init
    
    name: (kit1) react-starter-kit
    version: (1.0.0) 
    description: A React starter kit
    entry point: (index.js) src/main.jsx
    test command: 
    git repository: git@github.com:soulmachine/react-starter-kits.git
    keywords: es6,react,babel6,webpack,boilerplate
    author: soulmachine
    license: (ISC) MIT

It will generate a file `package.json`:

```json
{
  "name": "react-starter-kit",
  "version": "1.0.0",
  "description": "A React starter kit",
  "main": "src/main.jsx",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "es6",
    "react",
    "babel6",
    "webpack",
    "boilerplate"
  ],
  "author": "soulmachine",
  "license": "MIT"
}
```

## 1.1 Install React

    npm install --save react react-dom

## 1.2 Install Babel 6

Babel 6 changes drastically compared with Babel 5. Every transform is now a plugin, including ES2015 and JSX. Stage 0 is now a separate preset, not an option(used to be `stage: 0` in `.babelrc` in babel 5.x). So we need to install them addtionally.

	npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babel-preset-stage-0

And three presets to `.babelrc`:

    "presets": ["es2015", "react", "stage-0"]

**Runtime support**

Babel can’t support all of ES6 with compilation alone — it also requires some runtime support. In particular, the new ES6 built-ins like Set, Map and Promise must be polyfilled, and Babel’s generator implementation also uses a number of runtime helpers.

    npm install --save babel-polyfill

Babel also bakes a number of smaller helpers directly into your compiled code(used to be `--external-helpers` option in Babel 5.x). This is OK for single files, but when bundling with Webpack, repeated code will result in heavier file size. It is possible to replace these helpers with calls to the `babel-runtime` package by adding the `transform-runtime` plugin:

    npm install --save babel-runtime
    npm install --save-dev babel-plugin-transform-runtime

Add the plugin to `.babelrc`:

    "plugins": ["transform-runtime"]

Reference: 

+ [6.0.0 Released](http://babeljs.io/blog/2015/10/29/6.0.0/)
+ <http://babeljs.io/docs/setup/>
+ [The Six Things You Need To Know About Babel 6](http://jamesknelson.com/the-six-things-you-need-to-know-about-babel-6/)
+ [Using ES6 and ES7 in the Browser, with Babel 6 and Webpack](http://jamesknelson.com/using-es6-in-the-browser-with-babel-6-and-webpack/)

## 1.3 Install Webpack

For those who don't know what webpack is, please read [this tutorial](https://github.com/ruanyf/webpack-demos).

    npm install --save-dev webpack

Install essential webpack loaders(What is loader? See [official docs here](https://webpack.github.io/docs/loaders.html)):

    npm install --save-dev babel-loader css-loader style-loader url-loader 

Add these loaders to `webpack.config.js`:

```javascript
module: {
  loaders: [
	{ 
	  test: /\.css$/, 
	  include: path.resolve(__dirname, 'src'),
	  loader: 'style-loader!css-loader?modules' 
	},
    {
      test: /\.jsx?$/,
	  include: path.resolve(__dirname, 'src'),
      exclude: /node_modules/,
      loader: 'babel'
    },
	{ 
	  test: /\.(png|jpg)$/,
      loader: 'url-loader?limit=8192'
    }
  ]
}
```

Babel requires some helper code to be run before your application. To achieve this, add the `babel-polyfill` to the `entry` section. 

If you write a component in a `.jsx` file instead of `.js`, you must import or require it with the `.jsx` extension. How to import components without `.jsx` suffix? Add a `resolve` field to `webpack.config.js`:

```javascript
resolve: {
  //When requiring, you don't need to add these extensions
  extensions: ["", ".js", ".jsx"]
},
```

Finally we have a complete `webpack.config.js`:

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = {
  entry: [
    'babel-polyfill',
    path.resolve(__dirname, 'src/main.jsx')
  ],
  resolve: {
    //When requiring, you don't need to add these extensions
    extensions: ["", ".js", ".jsx"]
  },
  output: {
    path: __dirname + '/build',
    publicPath: '/',
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'style-loader!css-loader?modules'
      },
      {
        test: /\.jsx?$/,
        include: path.resolve(__dirname, 'src'),
        exclude: /node_modules/,
        loader: 'babel'
      },
      {
        test: /\.(png|jpg)$/,
        loader: 'url-loader?limit=8192'
      }
    ]
  }
};
```

## 1.4 Write our code

See the code in `kit1/src` directory.

## 1.5 Compile and run

Add a subcommand `build` to the `scripts` section of `package.json`,

    "build": "webpack -d",

Compile the whole project,

    npm run build

It will generate a file `bundle.js` in `build/` directory.

Copy the `index.html` to `build/`,

    cp src/index.html build/
    
Open `build/index.html` in a browser, you'll see it!

How to copy `index.html` to `build/` automatically? There is a way, let's do it.

Install the loader `file-loader`,

    npm install --save-dev file-loader
    
In `main.jsx` file add a require to `index.html` via `file-loader`,

    require('file?name=[name].[ext]!./index.html') // eslint-disable-line

We also need to copy `main.css` to `build/`,

    require('file?name=[name].[ext]!./main.css') // eslint-disable-line


# 2 Kit2: Kit1 + webpack-dev-server + ESLint

We're going to use more features of webpack, to make the development workflow more powerful.

## 2.1 webpack-dev-server

The [webpack-dev-server](https://webpack.github.io/docs/webpack-dev-server.html) is actually a node.js Express server that serves static files. 

    npm install --save-dev webpack-dev-server

Add a subcommand `start` in `scripts` section of `package.json`:

    "start": "webpack-dev-server --devtool source-map --inline --progress --colors --content-base build",

Now run `npm run start` and open <http://localhost:8080> in a browser.

## 2.2 open-browser-webpack-plugin

[open-browser-webpack-plugin](https://github.com/baldore/open-browser-webpack-plugin) opens a new browser tab when Webpack loads. Very useful if you're lazy and don't want to force yourself to open a new tab when Webpack is ready to play!

    npm install --save-dev open-browser-webpack-plugin

Add require to this plugin and add a new instance to the `plugins` field of `webpack.config.js`:

```javascript
var OpenBrowserPlugin = require('open-browser-webpack-plugin');
// ...
plugins: [
  new OpenBrowserPlugin({ url: 'http://localhost:8080' })
]
```

Now run `npm run start` you'll find that it will launch a browser to open <http://localhost:8080> automatically.

    npm run start


## 2.3 Hot Module Replacement

[Hot Module Replacement](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) (HMR) exchanges, adds, or removes modules while an application is running **without a page reload**.

There are [two ways](http://webpack.github.io/docs/webpack-dev-server.html#hot-module-replacement) to enable Hot Module Replacement with the webpack-dev-server.

1. Specify `--hot` and `--inline` in the command line

        $ webpack-dev-server --hot --inline

2. Modify` webpack.config.js`.

    + add `new webpack.HotModuleReplacementPlugin()` to the `plugins` field
    + add `webpack/hot/dev-server` and `webpack-dev-server/client?http://localhost:8080` to the `entry` field

The first way is much simpler. Just add a flag `--hot` to the subcommand `start`:

    "start": "webpack-dev-server --devtool source-map --hot --inline --progress --colors --content-base build",


## 2.4 ESLint and Airbnb's ESLint config

Install:

    npm install --save-dev eslint eslint-config-airbnb eslint-plugin-react eslint-plugin-import eslint-plugin-jsx-a11y

Add `.eslintrc`:

```json
{
  "env": {
    "node": true
  },
  "ecmaFeatures": {
    "jsx": true
  },
  "globals": {
    "React": true
  },
  "plugins": [
    "react"
  ],
  "extends": "airbnb",
  "rules": {
    "comma-dangle": 0,
    "no-console": 0,
    "id-length": 0,
    "react/prop-types": 0
  }
}
```

Add `.eslintignore`:

    build/**
    node_modules/**
    **/*.css
    **/*.html

Add a subcommand `lint` to the `scripts` field of `package.json`:

    "lint": "eslint 'src/**/*.@(js|jsx)'",

Run `npm run lint` to lint all source code.

Add eslint to git pre-commit hook, this way you will never commit in code that doesn't pass a check.

    npm install --save-dev precommit-hook

In addition to the `precommit-hook` package, this command will also 

+ adds two files `.jshintrc` and `.jshintignore` to current project
+ adds a `pre-commit` field to `package.json`
+ adds two subcommands `lint` and `validate` to the `scripts` field if none exist

The `lint` subcommand in `scripts` uses `eslint` instead of `jshint`, which means this starter kit doesn't use jshint at all, and this is true. 

Since eslint supports ES6 while jshint only has partial support for ES6, the obvious choice is eslint.

Delete two files `.jshintrc` and `.jshintignore`:

    rm .jshint*

Actually this starter kit is almost the same as [ruanyf/react-babel-webpack-boilerplate](https://github.com/ruanyf/react-babel-webpack-boilerplate).

Reference:

+ [5-step quick start guide to ESLint](http://codeutopia.net/docs/eslint/)
+ [A Comparison of JavaScript Linting Tools](http://www.sitepoint.com/comparison-javascript-linting-tools/)
+ [ruanyf/react-babel-webpack-boilerplate](https://github.com/ruanyf/react-babel-webpack-boilerplate)
+ [ruanyf/webpack-demos](https://github.com/ruanyf/webpack-demos)
+ [petehunt/webpack-howto](https://github.com/petehunt/webpack-howto)

# 3 Kit3: Kit2 + Redux

Install Redux related packages,

    npm install --save redux react-redux redux-thunk

Let's write a simple component named `Counter`, which is almost the same with the official example, [redux/examples/counter/](https://github.com/reactjs/redux/tree/master/examples/counter).

Copy five directories `actions`, `components`, `containers`, `reducers` and `store` from the official counter example to our `src` directory.

Rename `component/Counter.js` to `component/Counter.jsx`.

In `containers/Counter.js` change the line `import Counter from '../components/Counter'` to `import Counter from '../components/Counter.jsx'`.

In `components/App.jsx`, add the `Counter` component:

```javascript
import Counter from '../containers/Counter'
//...

render() {
  return (
    <div>
      <h1>Hello World</h1>
      <Counter />
    </div>
  )
}
```

In `main.jsx` we create a `store` and make the `Provider` as the root component:

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import App from './components/App.jsx'
import { Provider } from 'react-redux'
import configureStore from './store/configureStore'
require('file?name=[name].[ext]!./index.html')

const store = configureStore()

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('app')
)
```

Compile and run:

    npm install
    npm run build
    npm run start

It will open in a browser automatically.

At last, I need to point out the **Project Layout** specifically. In a `React + Redux` project, the state is in Redux single store, so there is no need for components to have their own state, which means **all components are stateless**. The `src/components` is for stateless components which are not aware of Redux, and the `src/containers` is for [smart components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.9esqxa1km) which are responsible to pass down state to dumb components as props.

References:

+ [Full-Stack Redux Tutorial - Tero Parviainen](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html)
+ [happypoulp/redux-tutorial](https://github.com/happypoulp/redux-tutorial)
+ [redux/examples/counter/](https://github.com/rackt/redux/tree/master/examples/counter)
+ [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005#.sd66r3cj6)
+ [Smart and Dumb Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.9esqxa1km)
+ [container vs component?](https://github.com/rackt/redux/issues/756)
+ [React 0.14：揭秘局部组件状态陷阱](http://zhuanlan.zhihu.com/FrontendMagazine/20416954)


# 4 Kit4: Kit3 + react-router

## 4.1 Use react-router

Now the text `Hello World` and `Counter` component huddle together in one page, which looks ugly, how about split them into separate pages?

We want the `Hello World` text to be displayed at the home URL '/' and the counter at '/counter'. Here react-router comes to help.

Install react-router,

    npm install --save history react-router

Creat a `Header` component, which is a stateless component:

```javascript
import React from 'react'
import {Link} from 'react-router'

export default () =>
  <div>
    <Link to="/">Home</Link>
    {' '}
    <Link to="counter">Counter</Link>
  </div>
```

Split `components/App.jsx` to two files, `containers/App.jsx` and `components/HelloWorld.jsx`.

`containers/App.jsx`:

```javascript
import React from 'react'
import Header from '../components/Header.jsx'

export default (props) =>
  <div>
    <Header />
    {props.children}
  </div>
```

`components/HelloWorld.jsx`:

```javascript
import React from 'react'
require('./HelloWorld.css')

export default () =>
  <h1>Hello World</h1>
```

Rename the file `App.css` to `HelloWorld.css`.

Create a new file `src/routes.jsx`:

```javascript
import React from 'react'
import { Route, IndexRoute } from 'react-router'
import App from './containers/App.jsx'
import HelloWorld from './components/HelloWorld.jsx'
import Counter from './containers/Counter'

const routes =
  <Route path="/" component={App}>
    <IndexRoute component={HelloWorld} />
    <Route path="counter" component={Counter} />
  </Route>

export default routes
```

Make `Router` as the root component in `main.jsx` and delete `import App`:

```javascript
import routes from './routes.jsx'
import { Router, hashHistory } from 'react-router'
//...
ReactDOM.render(
  <Provider store={store}>
    <Router history={hashHistory}>{routes}</Router>
  </Provider>,
  document.getElementById('app')
)
```

Compile and Run,

    npm install
    npm start

Everything works!

## 4.2 Remove `#` in URLs

You'll see a `#` in all URLs, this is because react-router uses `hashHistory` by default.

Normally we want URLs <http://www.example.com/about> instead of <http://www.example.com/#/about>, how to achive this? Use `browserHistory`.

Modify `main.jsx` a little bit:

```javascript
import { Router, browserHistory } from 'react-router'
//...

<Router history={browserHistory()}>{routes}</Router>
```

Run `npm start` and take a look in browser. Looks Good!

But wait. When you refresh the page <http://localhost:8080/counter> it will return error. It seems you can only visit the counter page by clicking the link, why?

Why? Because for now react-router only works in browser, when you click fresh or hit enter on the browser address bar the request will be sent to server. In our web app we only have a simple express server bundled in `webpack-dev-server`, which doesn't understand the URL <http://localhost:8080/counter> at all. How to make our server understand URLs? There are several options:

+ run react-router at server-side
+ always return `index.html` and leave all work to client-side react-router

To keep our web app simple, we'll use the second way:

+ In development phase, make `webpack-dev-server` always return `index.html`
+ In deploy phase, make our HTTP server such as Nignx always return `index.html`

Add a flag `--history-api-fallback` to the subcommand `start` in `package.json`, then run `npm start` again and refresh at <http://localhost:8080/counter> or enter it directly in browser address bar, you'll see everything works!


# 5 Kit5: Kit4 + Mocha

[Mocha](https://mochajs.org/) is a JavaScript unit test framework, [Chai](http://chaijs.com/) is an assertion/expectation library.

    npm install --save-dev mocha chai

We're going to test our React components as well, and that's going to require a DOM. One alternative would be to run tests in an actual web browser with a library like [Karma](http://karma-runner.github.io/0.13/index.html). However, we don't actually need to do that because we can [get away with](http://jaketrent.com/post/testing-react-with-jsdom/) using [jsdom](https://github.com/tmpvar/jsdom), a pure JavaScript DOM implementation that runs in Node:

    npm install --save-dev jsdom

We also need a little setup code for jsdom before it's ready for React to use. 

+ We essentially need to create the `document` and `window` objects that would normally be provided by the web browser. Then we need to put them on the [global object](https://nodejs.org/api/globals.html#globals_global), so that they will be discovered by React when it accesses `document` or `window`. 
+ Additionally, we need to take all the properties that the `window` object contains, such as `navigator`, and hoist them on to the Node.js global object, so that properties provided by `window` can be used without the `window.` prefix, which is what would happen in a browser environment. Some of the code inside React relies on this

We can put this kind of setup code in a file `test/test_helper.js`:

```javascript
import jsdom from 'jsdom'
import chai from 'chai'

const doc = jsdom.jsdom('<!doctype html><html><body></body></html>')
const win = doc.defaultView

global.document = doc
global.window = win

// from mocha-jsdom https://github.com/rstacruz/mocha-jsdom/blob/master/index.js#L80
Object.keys(window).forEach((key) => {
  if (!(key in global)) {
    global[key] = window[key];
  }
})
```

We need to run this file before any unit tests. To achive this, we need to add a flag `--require ./test/test_helper.js` to the `mocha` command.

To enable ES6 syntax in test code, we need to:

1. Use Babel to transpile ES6 code before running them

    To achive this, add a flag `--compilers js:babel-core/register`

1. Add `import 'babel-polyfill'` to `test/test_helper.js`

    Babel won't transpile global objects such as `Iterator`, `Generator`, `Promise`, `Map`, `Set`, so we need to polyfill them in all test code. And the most convenient way is to add it to `test/test_helper.js`.

After figuring out all flags needed by `mocha` command, we can modify the `test` subcommand in the `scripts` field of `package.json`:

    "test": "mocha --compilers js:babel-core/register --require ./test/test_helper.js 'test/**/*.@(js|jsx)'"

At last let's write a simple unit test case. First create a file `test/add.js`:

```javascript
export default function add(x, y) {
  return x + y
}
```

Second, write a unite test file `test/add.test.js`:

```javascript
import add from './add'
import { expect } from 'chai'

describe('add function', function() {
  it('1 plus 1 equal to 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  })
})
```

Type "`npm test`" to run all unit tests.

References:

+ [测试框架 Mocha 实例教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
+ [Unit Testing support](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html#unit-testing-support)
+ [Testing in ES6 with Mocha and Babel 6](http://jamesknelson.com/testing-in-es6-with-mocha-and-babel-6/)


# 6 Kit6: Kit5 + material-ui

    npm install --save material-ui react-tap-event-plugin

Let's replace the `+` button in the `Counter` component to [RaisedButton](http://www.material-ui.com/#/components/buttons). 

In ``src/components/Counter.jsx` import the `RaisedButton` and replace the `+` button with it:

```jsx
import RaisedButton from 'material-ui/RaisedButton'
//...
<RaisedButton label="+" secondary={true} onTouchTap={increment}/>
```

Change the `<p>` to `<div>` or there will be warnings in console.

Add the following lines to `src/main.jsx`:

```javascript
import injectTapEventPlugin from 'react-tap-event-plugin'

// Needed for onTouchTap
// http://stackoverflow.com/a/34015469/988941
injectTapEventPlugin()
```

Modify the `src/containers/App.jsx` to the following:

```jsx
import React from 'react'
import getMuiTheme from 'material-ui/styles/getMuiTheme'
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider'
import Header from '../components/Header'

export default (props) => (
  <MuiThemeProvider muiTheme={getMuiTheme()}>
    <div>
      <Header />
      {props.children}
    </div>
  </MuiThemeProvider>
)
```

Now run `npm start` and you'll see the new button in browser.


# 7 Kit7: Kit6 + redux-devtools

## 7.1 Installation

    npm install --save-dev redux-devtools redux-devtools-log-monitor redux-devtools-dock-monitor
    
## 7.2 Create a `DevTools` Component

`src/containers/DevTools.jsx`:

```jsx
import React from 'react'

// Exported from redux-devtools
import { createDevTools } from 'redux-devtools'

// Monitors are separate packages, and you can make a custom one
import LogMonitor from 'redux-devtools-log-monitor'
import DockMonitor from 'redux-devtools-dock-monitor'

// createDevTools takes a monitor and produces a DevTools component
const DevTools = createDevTools(
  // Monitors are individually adjustable with props.
  // Consult their repositories to learn about those props.
  // Here, we put LogMonitor inside a DockMonitor.
  // Note: DockMonitor is visible by default.
  <DockMonitor
    toggleVisibilityKey="ctrl-h"
    changePositionKey="ctrl-q"
    defaultIsVisible
  >
    <LogMonitor theme="tomorrow" />
  </DockMonitor>
)

export default DevTools
```

## 7.3 Use `DevTools.instrument()` Store Enhancer

Modify the file `src/store/configureStore.js`:

```javascript
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import rootReducer from '../reducers'
import DevTools from '../containers/DevTools'

const enhancer = compose(
  // Middleware you want to use in development:
  applyMiddleware(thunk),
  // Required! Enable Redux DevTools with the monitors you chose
  DevTools.instrument()
)

export default function configureStore(initialState) {
  // Note: only Redux >= 3.1.0 supports passing enhancer as third argument.
  // See https://github.com/rackt/redux/releases/tag/v3.1.0
  const store = createStore(rootReducer, initialState, enhancer)

  // Hot reload reducers (requires Webpack or Browserify HMR to be enabled)
  if (module.hot) {
    module.hot.accept('../reducers', () =>
      store.replaceReducer(require('../reducers').default) // eslint-disable-line
    )
  }

  return store
}
```

## 7.4 Render `<DevTools>` in Your App

Add `DevTools` to `src/containers/App.jsx`:

```jsx
import React from 'react'
import Header from './Header'
import DevTools from '../containers/DevTools'

export default (props) =>
  <div>
    <Header />
    {props.children}
    <DevTools />
  </div>
```

Run `npm start` and you'll see a `DevTools` panel on the right side. Try to click the `+` button and you'll see that `DevTools` is tracking the changes of state.


## 7.5 Exclude DevTools from Production Builds


### 7.5.1 Split `store/configureStore.js` to three files

`store/configureStore.js`:

```javascript
// Use ProvidePlugin (Webpack) or loose-envify (Browserify)
// together with Uglify to strip the dev branch in prod build.
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./configureStore.production')
} else {
  module.exports = require('./configureStore.development')
}
```

`store/configureStore.production.js`:

```javascript
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import rootReducer from '../reducers'

const enhancer = applyMiddleware(
  thunk
)

export default function configureStore(initialState) {
  // Note: only Redux >= 3.1.0 supports passing enhancer as third argument.
  // See https://github.com/rackt/redux/releases/tag/v3.1.0
  return createStore(rootReducer, initialState, enhancer)
}
```

`store/configureStore.development.js`:

```javascript
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import rootReducer from '../reducers'
import DevTools from '../containers/DevTools'

const enhancer = compose(
  // Middleware you want to use in development:
  applyMiddleware(thunk),
  // Required! Enable Redux DevTools with the monitors you chose
  DevTools.instrument()
)

export default function configureStore(initialState) {
  // Note: only Redux >= 3.1.0 supports passing enhancer as third argument.
  // See https://github.com/rackt/redux/releases/tag/v3.1.0
  const store = createStore(rootReducer, initialState, enhancer)

  // Hot reload reducers (requires Webpack or Browserify HMR to be enabled)
  if (module.hot) {
    module.hot.accept('../reducers', () =>
      store.replaceReducer(require('../reducers').default) // eslint-disable-line
    )
  }

  return store
}
```


### 7.5.2 Create a `DevTools` only visible in development mode

`containers/DevTools.js`:

```jsx
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./DevTools.production')
} else {
  module.exports = require('./DevTools.development')
}
```

`containers/DevTools.production.jsx`:

```jsx
import React from 'react'

export default () => <div></div>
```

Rename `containers/DevTools.jsx` to `containers/DevTools.development.jsx`:


### 7.5.3 Split `webpack.config.js` to three files

`webpack.config.base.js`:

```javascript
'use strict';

var path = require('path');

module.exports = {
  entry: [
    'babel-polyfill',
    path.resolve(__dirname, 'src/main.jsx')
  ],
  resolve: {
    //When requiring, you don't need to add these extensions
    extensions: ["", ".js", ".jsx"]
  },
  output: {
    path: __dirname + '/build',
    publicPath: '/',
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'style-loader!css-loader?modules'
      },
      {
        test: /\.jsx?$/,
        include: path.resolve(__dirname, 'src'),
        exclude: /node_modules/,
        loader: 'babel'
      },
      {
        test: /\.(png|jpg)$/,
        loader: 'url-loader?limit=8192'
      }
    ]
  }
};
```

`webpack.config.production.js`:

```javascript
'use strict';

var webpack = require('webpack');
var baseConfig = require('./webpack.config.base');

var config = Object.create(baseConfig);
config.plugins = [
  new webpack.optimize.OccurenceOrderPlugin(),
  new webpack.optimize.DedupePlugin(),
  new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  new webpack.optimize.UglifyJsPlugin({
    compressor: {
      screw_ie8: true,
      warnings: false
    }
  })
];

module.exports = config;
```

`webpack.config.development.js`:

```javascript
'use strict';

var webpack = require('webpack');
var baseConfig = require('./webpack.config.base');
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

var config = Object.create(baseConfig);
config.plugins = [
  new webpack.HotModuleReplacementPlugin(),
  new OpenBrowserPlugin({ url: 'http://localhost:8080' }),
  new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('development')
  })
];

module.exports = config;
```


### 7.5.4 Add a new build command to `package.json`

Add a new build command to `scripts` field of `package.json`:

    "build:production": "webpack -p --config webpack.config.production.js",

We also need to specify the configuration file for the `build` and `start` subcommands:

    "build": "webpack -d --config webpack.config.development.js",
    "start": "webpack-dev-server --devtool source-map --hot --inline --progress --colors --history-api-fallback --config webpack.config.development.js --content-base build",


### 7.5.5 Run and Verify

    npm run build:production
    cd build
    # check the size of bundle.js
    ls -lh
    python -m SimpleHTTPServer 8080

Open <http://localhost:8080/> in a browser you'll see that the DevTools panel is gone.

Besides, the size of `bundle.js` becomes much smaller now.
