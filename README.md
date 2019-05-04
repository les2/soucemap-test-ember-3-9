# soucemap-test-ember-3-9

Adding the [ember-tooltips] addon to your Ember project has the effect of "breaking" the sourcemaps for the vendor bundle.

By "break" I mean that the `sourcesContent` element of the souremaps will be omitted. This can cause tooling, such as [New Relic Browser] and [Sentry], to fail to render sourcemaps properly in Error reports.

The problem appears to be caused by the `app.import(` calls for `popper.js` and `tooltip.js` in the `included(` Ember addon hook of the [ember-tooltips] addon:

```
  included: function(app) {
    this._super.included.apply(this, arguments);

    app.import("node_modules/popper.js/dist/umd/popper.js", {
      using: [
        {
          transformation: "amd",
          as: "popper.js"
        }
      ]
    });

    app.import("node_modules/tooltip.js/dist/umd/tooltip.js", {
      using: [
        {
          transformation: "amd",
          as: "tooltip.js"
        }
      ]
    });
  }
```

When building, notice that [broccoli-uglify-sourcemap] logs the following warning to the logs:

```
⠇ Building[WARN] (broccoli-uglify-sourcemap) "popper.js.map" referenced in "assets/vendor.js" could not be found
```

## Steps to Reproduce

```
$ ember build --environment production
$ jq '.sourcesContent' dist/assets/vendor-*.map
null
```

If that prints `null`, then the vendor sourcemap is missing the `sourcesContent` element.

Now remove the `ember-tooltips` addon from `package.json` and build again:
```
$ yarn remove ember-tooltips && yarn
$ ember build --environment production
$ jq '.sourcesContent' dist/assets/vendor-*.map
[
  "window.EmberENV = {\"FEATURES\":{},\"EXTEND_PROTOTYPES\":{\"Date\":false}};\nvar runningTests = false;\n\n\n",
...
```

Lots of sourcecode should be printed, showing that the addition of [ember-tooltips] is what breaks the sourcemaps.

[ember-tooltips]: https://github.com/sir-dunxalot/ember-tooltips
[New Relic Browser]: https://docs.newrelic.com/docs/browser/new-relic-browser/browser-pro-features/upload-source-maps-un-minify-js-errors
[Sentry]: https://docs.sentry.io/platforms/javascript/sourcemaps/
[broccoli-uglify-sourcemap]: https://github.com/ember-cli/broccoli-uglify-sourcemap

