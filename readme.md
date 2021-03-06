# Osiris and OJS

## OJS is an asynchronous Javascript template engine.
Designed to build static sites or be used with express.

Basic template syntax:
```javascript
<?
// we start our javascript content with <? and end it with ?>
// we can use <?= ?> to print a statement
// anything javascript goes, here's a test function
const myFunction = async () => { // async lets us await
  await print('<p>Hi from myFunction</p>'); // print is the only function available with OJS without Osiris
};
?>
<!DOCTYPE html>
<html>
<head>
  <title><?='Hi from Javascript!' ?></title>
</head>
<body>
<?
await print('<p>We must await our print statements</p>');
await print('<p>Everything is asynchronous here</p>');
await myFunction(); // we can await our own functions too
?>
</body>
</html>
```

## Osiris is a framework built on top of OJS
Designed to facility code re-use and organisation of files

- [Example build script](https://github.com/seam-project-studios/osiris-ojs/blob/master/build.js)
- [Example build and host static](https://github.com/seam-project-studios/osiris-ojs/blob/master/static.js)
- [Example express hook](https://github.com/seam-project-studios/osiris-ojs/blob/master/dev.js)

## Basic Osiris build example
```javascript
const osiris = require('osiris-ojs'); // renderer

// we can inject things into scope to be used by all renderings
osiris.use({
  aGlobalFunction: async () => {
    await this.print('Hi from aGlobalFunction');
  }
});

let writeFile = fs.createWriteStream('myBuilt.html');
await osiris.render(writeFile, 'myToBuild.ojs', {
  myLocalFunction: async () => { // we can inject things just for this rendering
    await this.print('Hi from myLocalFunction');
  }
});
```

## Osiris API
Osiris exposes the following functions to the templates, as well as the default `print()` provided by OJS
- `q: async? (str='')`, translates a strings HTML entities so it can be used within quoted attributes, returns a promise if given a promise
- `snippet: async (filename, args)`, renders filename found in src/snippets/`filename`.ojs with the arguments provided as `args` to the template
- `element: async (filename, args)`, renders filename found in src/elements/`filename`.ojs with the arguments provided as `args` to the template
- `js: (str)`, bundles collections of Javascript for footer insertion
- `css: (str)`, bundles collections of CSS for footer insertion
- `bundleJs`, retrieves the JS bundle for output
- `bundleCss`, retrieves the CSS bundle for output
- `onClose`, this callback is provided by OJS for unexpected user connection closing, for clean up tasks that may need to be done

## Osiris express
Osiris comes with hooks to get to the most basic functionality within express, usage:
```javascript
const HTTP_PORT = 8080;
const osiris = require('osiris-ojs');
const ojsExpress = require('osiris-ojs/express');

const express = require('express');
const app = express();
const main = async () => {
  app.use(async (req, res, next) => { // anything not served lands here
    let filename = req.path.substr(1); // trim starting /

    if (filename === '') {
      filename = 'index'; // default page for directory index
    }

    if (!await fs.exists('./src/pages/' + filename + '.ojs')) {
      return next(); // file doesn't exist, bail
    }

    res.header('content-type', 'text/html'); // we have something

    // call renderer with our addons, we can block here with await if we need any clean up after render
    await osiris.render(res, './src/pages/' + filename + '.ojs', {
      express: ojsExpress(req, res) // this gives templates access to get, post, header() and headersSent
    });
    // render complete, res.end() sent, clean up
  });
  await app.listen(HTTP_PORT, () => {
    console.log('Node process listening on ' + HTTP_PORT);
  });
};

main();
```
### Osiris express API
- `get`, object containing get variables, parsed by `url.parse`
- `post`, object containing post variables, taken from `req.body`
- `this.header = (...args)`, calls `res.header`
- `headersSent`, boolean if headers have been sent yet
- `cookie`, object containing cookie variables, taken from `req.cookies`
- `setCookie = (...args)`, calls `res.cookie`
