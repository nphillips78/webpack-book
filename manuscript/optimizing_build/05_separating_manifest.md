# Separating a Manifest

When webpack writes bundles, it maintains a **manifest** as well. You can find it in the generated *vendor* bundle in this project. The manifest describes what files webpack should load. It is possible to extract it and start loading the files of our project faster instead of having to wait for the *vendor* bundle to be loaded.

This is the root of our problem. If the hashes webpack generates change, then the manifest will change as well. As a result, the contents of the vendor bundle will change and it will become invalidated. The problem can be eliminated by extracting the manifest to a file of its own or by writing it inline to the *index.html* of the project.

T> To understand how a manifest is generated in greater detail, [read the technical explanation at Stack Overflow](https://stackoverflow.com/questions/39548175/can-someone-explain-webpacks-commonschunkplugin/39600793).

## Extracting a Manifest

We have done most of the work already when we set up `extractBundles`. To extract the manifest, a single change is required:

**webpack.config.js**

```javascript
...

const productionConfig = merge([
  ...
  parts.extractBundles({[
      {
        name: 'vendor',
        minChunks: ({ context }) => (
          context && context.indexOf('node_modules') >= 0
        ),
      },
leanpub-start-insert
      {
        name: 'manifest',
      },
leanpub-end-insert
  ]),
  ...
]);

...
```

If you build the project now (`npm run build`), you should see something like this:

```bash
Hash: 940612921489b7d36dc2
Version: webpack 2.2.1
Time: 4227ms
                  Asset       Size  Chunks             Chunk Names
            136bdd5b.js    1.47 kB       3  [emitted]  manifest
           674f50d2.eot     166 kB          [emitted]
         af7ae505.woff2    77.2 kB          [emitted]
          fee66e71.woff      98 kB          [emitted]
           11ec0064.svg   22 bytes          [emitted]
           85011118.png      77 kB          [emitted]
    scripts/2ef8cb5e.js  184 bytes    0, 3  [emitted]
            4f2ad811.js    19.6 kB    1, 3  [emitted]  vendor
            3a05df06.js  717 bytes    2, 3  [emitted]  app
           b06871f2.ttf     166 kB          [emitted]
           03d7fa6d.css  164 bytes    2, 3  [emitted]  app
           9c2469e8.css    46.6 kB    1, 3  [emitted]  vendor
scripts/2ef8cb5e.js.map  824 bytes    0, 3  [emitted]
        4f2ad811.js.map     247 kB    1, 3  [emitted]  vendor
       9c2469e8.css.map   89 bytes    1, 3  [emitted]  vendor
        3a05df06.js.map    5.99 kB    2, 3  [emitted]  app
       03d7fa6d.css.map   89 bytes    2, 3  [emitted]  app
        136bdd5b.js.map      14 kB       3  [emitted]  manifest
             index.html  387 bytes          [emitted]
   [4] ./~/object-assign/index.js 2.11 kB {1} [built]
  [14] ./app/component.js 372 bytes {2} [built]
  [15] ./app/shake.js 138 bytes {2} [built]
...
```

This simple change gave us a separate file that contains the manifest. In the output above it has been marked with `manifest` chunk name. Because we are using *html-webpack-plugin*, we don't have to worry about loading the manifest ourselves as the plugin adds a reference to *index.html* for us.

Plugins, such as [inline-manifest-webpack-plugin](https://www.npmjs.com/package/inline-manifest-webpack-plugin) and [html-webpack-inline-chunk-plugin](https://www.npmjs.com/package/html-webpack-inline-chunk-plugin), [assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin), work with *html-webpack-plugin* and allow you to write the manifest within *index.html* in order to avoid a request.

T> To get a better idea of the manifest contents, comment out `parts.minify()` and examine the resulting manifest. You should see something familiar there.

Try adjusting *app/index.js* and see how the hashes change. This time around it should **not** invalidate the vendor bundle, and only the manifest and app bundle names should be different like this:

```bash
Hash: e30a9f69eec18c5fe1c8
Version: webpack 2.2.1
Time: 3849ms
                  Asset       Size  Chunks             Chunk Names
            3df3e08c.js    1.47 kB       3  [emitted]  manifest
           674f50d2.eot     166 kB          [emitted]
         af7ae505.woff2    77.2 kB          [emitted]
          fee66e71.woff      98 kB          [emitted]
           11ec0064.svg   22 bytes          [emitted]
           85011118.png      77 kB          [emitted]
    scripts/2ef8cb5e.js  184 bytes    0, 3  [emitted]
            4f2ad811.js    19.6 kB    1, 3  [emitted]  vendor
            67a40f61.js  737 bytes    2, 3  [emitted]  app
           b06871f2.ttf     166 kB          [emitted]
           03d7fa6d.css  164 bytes    2, 3  [emitted]  app
           9c2469e8.css    46.6 kB    1, 3  [emitted]  vendor
scripts/2ef8cb5e.js.map  824 bytes    0, 3  [emitted]
        4f2ad811.js.map     247 kB    1, 3  [emitted]  vendor
       9c2469e8.css.map   89 bytes    1, 3  [emitted]  vendor
        67a40f61.js.map    6.06 kB    2, 3  [emitted]  app
       03d7fa6d.css.map   89 bytes    2, 3  [emitted]  app
        3df3e08c.js.map      14 kB       3  [emitted]  manifest
             index.html  387 bytes          [emitted]
   [4] ./~/object-assign/index.js 2.11 kB {1} [built]
  [14] ./app/component.js 372 bytes {2} [built]
  [15] ./app/shake.js 138 bytes {2} [built]
...
```

T> In order to integrate with asset pipelines, you can consider using plugins like [chunk-manifest-webpack-plugin](https://www.npmjs.com/package/chunk-manifest-webpack-plugin), [webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin), [webpack-assets-manifest](https://www.npmjs.com/package/webpack-assets-manifest), or [webpack-rails-manifest-plugin](https://www.npmjs.com/package/webpack-rails-manifest-plugin). These solutions emit JSON that maps the original asset path to the new one.

T> One more way to improve the build further would be to load popular dependencies, such as React, through a CDN. That would decrease the size of the vendor bundle even further while adding an external dependency on the project. The idea is that if the user has hit the CDN earlier, caching can kick in just like here.

## Using Records

As mentioned in the *Splitting Bundles* chapter, plugins such as `AggressiveSplittingPlugin` use **records** to implement caching. The approaches discussed above are still valid, but records go one step further.

Records are used for storing module IDs across separate builds. The gotcha is that you need to store this file some way. If you build locally, one option is to include it to your version control.

To generate a *records.json* file, adjust the configuration as follows:

**webpack.config.js**

```javascript
...

const productionConfig = merge([
  {
    ...
leanpub-start-insert
    recordsPath: 'records.json',
leanpub-end-insert
  },
  ...
]);

...
```

If you build the project (`npm run build`), you should see a new file, *records.json*, at the project root. The next time webpack builds, it will pick up the information and rewrite the file if it has changed.

Records are particularly useful if you have a complex setup with code splitting and want to make sure the split parts gain correct caching behavior. The biggest problem is maintaining the record file.

T> `recordsInputPath` and `recordsOutputPath` give more granular control over input and output, but often setting only `recordsPath` is enough.

## Conclusion

Our project has basic caching behavior now. If you try to modify *app.js* or *component.js*, the vendor bundle should remain the same. But what's contained in the build? You can figure that out by *Analyzing Build Statistics*, as we'll do in the next chapter.
