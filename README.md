chrome-app-module-loader
========================

A module loader that lets you load npm modules from a chrome app.

To bootstrap, load `module.js` in a script tag.

```html
<script src="module.js" moduledir="/deps/" autorequire="/app/main.js"></script>
```

The `moduledir` attribute tells the loader where to load all package paths (paths that don't start with `/` or `.`).  If it's left out, the loader will assume you used npm and look for ./node_modules, ../node_modules, ...

The `autorequire` attribute tells it what file to require first.  It will `require.async` this module and load all it's dependencies as soon as the module system is initialized.

If you want use the module system your background page, just create the script tag using js.

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
