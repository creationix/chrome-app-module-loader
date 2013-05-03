chrome-app-module-loader
========================

A module loader that lets you load npm modules from a chrome app.

To bootstrap, load `module.js` in a script tag.

```html
<script src="module.js" moduledir="/deps/" autorequire="/app/main.js"></script>
```

The `moduledir` attribute tells the loader where to load all package paths (paths that don't start with `/` or `.`).  If it's left out, the loader will assume you used npm and look in `./node_modules/`, `../node_modules/`, etc...

The `autorequire` attribute tells it what file to require first.  It will `require.async` this module and load all it's dependencies as soon as the module system is initialized.

If you want use the module system your background page, just create the script tag manually using JavaScript.

```js
(function () {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.setAttribute('autorequire', "/app/main.js");
  script.setAttribute('moduledir', "/deps/");
  script.src = "/deps/chrome-app-module-loader/module.js";
  document.head.appendChild(script);
}());
```

## Modules

Modules are authored using the commonJS format.  This means you file will be wrapped in a function.  You can declare top-level local variables and they won't leak to other modules.  Anything you want to export needs to be put on the `exports` object.  Alternatively you can outright replace `module.exports` with a new object or function.

```js
// An example exporting a single function
module.exports = function (config) {
  // code goes here
  return {
    // more code goes here
  };
}
```

## Tips

Since the loader loads your original scripts using XHR, it's best if it can guess the filenames on the first try.  A couple tips make this work better.

 - Always include the file extension when loading a plain `.js` file.  For example `require('./helper.js')` instead of `require('./helper')`.
 - When loading a package, never include the extension.
 - Prefer `package.json` over `index.js` in packages.
 - Use the `moduledir` directive if you want a flat dependency tree.
 - Otherwise make sure every dependency is nested.

## Folder Resolution Rules

There are a few general ways to require a module.

### Relative Require

A relative require is one in which the path begins with the `.` character.  It's path will be resolved relative to the javascript file that made the require call.

### Absolute Require

An absolute require is one in which the path begins with `/`.  These are loaded relative to your domain root.  This is usually where your `manifest.json` is located.

### Dependency Require

A path that starts out with any other character is considered an external dependency and looks either in the global `moduledir` folder or recursively searches for `node_modules` folders in node.js style relative to the calling file.

### Package Require

If you include the `.js` extension, no further changes are made, it looks directly for that file.

When you leave the extension off your require path, the loader assumes you mean to load a package.  After resolving the directory using one of the three previous techniques, it looks for the actual file using this set of rules.

If you have no extension, it first assumes the name is a folder and looks for a package.json in that folder.  If found, it parses it looking for the `main` field.  If there is a main field, it loads that file as a relative require to the package.json.

If there is no package.json, it looks for a file called `index.js` in the assumed folder.  If that exists, it is the resolved path.

Lastly, as a fallback, it will append the `.js` extension to your original path and see if you just forgot to include it.

