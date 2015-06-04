# webpack-start
Introduction to Webpack tutorial

## Quick Start

* Checkout this repository
* run `npm install`

## Work through the tutorial

The following is a simple walk-through that will get you familiar with the basics of Webpack. The tutorial should take about fifteen minutes.

1. Open index.html in the file in the browser.
2. Open the browser developer tools, and observe that the browser returns a 404 when trying to get `bundle.js`.
3. Let's build `bundle.js` using Webpack. In a terminal, type `webpack ./entry.js bundle.js`.
4. Refresh the browser. You should see the words "Hello, world." in the browser.
5. Open `entry.js` in your editor. Change "world" to "Workfront" and save the file.
6. In the terminal, run `webpack ./entry.js bundle.js` again and refresh the browser. You should see "Hello, Workfront.".
7. Manually running the webpack command after every change is tedious. Run `webpack -w ./entry.js bundle.js`, where `-w` is the watch flag. When your entry file changes or any of its dependencies, it will rebuild the bundle. 
8. Let's try out the watcher. Edit `entry.js` so that it ends the sentence with a "!" instead of a ".". Save the file. You should see in the terminal that it saw that your file changed and rebuilt by the bundle. Refresh your browser to see the change.
9. So far, we've been using Webpack as a glorified file copier. Let's add a dependency to our `entry.js` file and see what Webpack does. Change `entry.js` as follows:

```js
'use strict';
document.write('Hello, Workfront! The current time is: ' + require('./displayTime'));
```

We are including `displayTime.js` as a CommonJS dependency in our file. Open up `displayTime.js`. You'll see the following:

```js
define(['moment'], function(moment) {
  return moment().format('dddd, MMMM D, YYYY, h:mm:ss a');
});
```

This file defines an AMD module. Webpack natively understands both CommonJS and AMD syntax. Additionally, it uses a third party module, moment, which is a date parsing, formatting, and localization library. Where does it get this dependency from? It actually looks in your node_modules directory for any dependencies. Moment was installed as a npm dependency.

Refresh your browser, and you'll see the changes that you made, along with the current date and time.

10. Open `bundle.js` in your editor. While the file is fairly easy to read, it has a lot of extra Webpack noise that makes it difficult to debug in the browser. Let's add source maps. Stop the Webpack watch that you're currently running, and instead run `webpack -w --devtool source-map ./entry.js bundle.js`. The new switch, `--devtool`, will generate a source map as part of the build. This will make debugger easier.
11. Refresh the browser, and take a look at the browser dev tools "source" section. You'll see a root named "webpack://". Expand the source tree there and you'll find the source for both "./entry.js" and "./displayTime.js". 
12. Let's try debugging the application using Webpack and the source maps. Add a `debugger;` statement in `displayTime.js`, like so:

```js
define(['moment'], function(moment) {
  debugger;
  return moment().format('dddd, MMMM D, YYYY, h:mm:ss a');
});
```

13. Refresh the browser, and notice that the app stops at the debugger breakpoint. Convenient, huh? Hit the continue button to let the page continue. Once you're done playing with the debugger, remove the debugger statement from the code.
14. Our page looks kind of plain. Let's add some style. Create a file, `style.css`, in the root of your project and add the following content to it:

```css
body {
  font-size: 2em;
  background-color: greenyellow;
}
```

15. How do we use this new stylesheet? Let's require it in `entry.js`:

```js
'use strict';
require('./style.css');
// ... rest of file
```

16. Save `entry.js`. You'll see an error in your terminal similar to the following:

```
ERROR in ./style.css
Module parse failed: /Users/jeremylund/dev/training/webpack-start/style.css Line 1: Unexpected token {
You may need an appropriate loader to handle this file type.
| body {
|   font-size: 2em;
|   background-color: greenyellow;
 @ ./entry.js 2:0-22
```

By default, Webpack treats every file as JavaScript. Since this isn't JavaScript, it doesn't know what to do with it. Thankfully, the error message gives us a hint as to what to do. We need to install a loader. Loaders are used to transform files into a form that Webpack can handle.

17. Stop the Webpack watcher, and type the following in the terminal: `npm install style-loader css-loader -D`. This will install the two loaders that we need. Modify the where we are requiring the stylesheet in `entry.js` to the following:

```js
require('style!css!./style.css');
```

We are using two loaders, and they work like a pipeline. First, it applies the css-loader (loaders working right-to-left), which transforms CSS into a string, and the style-loader applies the style to the page. Start your Webpack watch command back up, refresh the browser, and you should see the style in the browser. Change some of the style rules, and watch how Webpack rebuilds the bundle with every change. You can refresh the browser to see the changes.

18. While it was convenient to add the loaders right to the require function call, it kind of deviates from what a normal require call. Let's add a Webpack configuration file to handle the loaders. This will also give us an opportunity to remove some of the switches that we are using from the command line. Create a `webpack.config.js` file in the root of your project, and add the following content to it:

```js
module.exports = {
  entry: './entry.js',
  output: {
    filename: 'bundle.js'
  },
  devtool: 'source-map'
};
```

Run `webpack -w`, and everything should still work.

19. Of course, we added the config file so that we don't have to put the loaders on every CSS file. Add a module loaders section to `webpack.config.js`:

```js
module.exports = {
  entry: './entry.js',
  output: {
    filename: 'bundle.js'
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css!'}
    ]
  }
};
```

The loaders section defines an array of loaders to apply when the filename matches the test regular expression. The rule we just added will apply to the css-loader and style-loader to every CSS file it includes. After you make this change, remove the loaders from the require() call, then restart `webpack -w` to apply the changes to the Webpack configuration.

20. One thing that has been rather annoying to do is refresh the browser after any change. Using the Webpack Dev Server, your changes can not only rebuilt by Webpack, but also pushed to the browser. This is an easy switch.
21. Stop any Webpack watcher that is currently running.
22. In the terminal, run `webpack-dev-server --inline --hot`. This will start a web server on port 8080. The `--inline` flag tells Webpack to inline the hot reload client in the bundle, and the `--hot` flag enabled hot reloading.
23. Close the browser window you've been using, and instead navigate to `http://localhost:8080`. You should see the same content that we've already been working with.
24. Make some changes to the the stylesheet, to the text in `entry.js`, or any other changes. As soon as you save any of these files, notice how Webpack Dev Server re-builds the bundle and pushes the changes to the browser. Very nice!
