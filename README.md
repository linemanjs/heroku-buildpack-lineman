**Update 9/30/2015:** This buildpack will cease to work on 11/4/2015 when Heroku switches to cedar-14. Please migrate to cedar-14 by using @hone's static buildpack. Here is an [example Lineman project using his buildpack](https://github.com/hone/lineman.js-on-Heroku).


Old docs continue:


Heroku buildpack: Lineman
=========================

## Not using this buildpack

** You probably don't need this buildpack **

Unless you're using Sass, this buildpack is probably not worth attempting to use. Use the default Node.js buildpack with a custom run script instead.


To do this, configure the official Node.js buildpack and then enable the installation of development dependencies:

``` bash
$ heroku config:set BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-nodejs
$ heroku config:set NPM_CONFIG_PRODUCTION=false
```

Then, add a `Procfile` to your repo root if it doesn't already exist:

``` Procfile
web: npm run production
```

Finally, define the production script if it wasn't already defined by your lineman archetype:

``` json
  "scripts": {
    "start": "lineman run",
    "test": "lineman spec-ci",
    "production": "lineman clean build && npm i express@3 && node -e \"var e = require('express'), a = e(); a.use(e.static('dist/')); a.listen(process.env.PORT)\""
  },
```


## Using this buildpack


This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for [Lineman](https://github.com/linemanjs/lineman) apps.

Usage
-----

Example usage:

    $ heroku create --stack cedar --buildpack http://github.com/linemanjs/heroku-buildpack-lineman.git

Or, for an existing heroku app:

    $ heroku config:set BUILDPACK_URL=http://github.com/linemanjs/heroku-buildpack-lineman.git

And the output will look like this when you push to heroku:

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Lineman app detected
    -----> Vendoring node 0.4.7
    -----> Installing dependencies with npm ....
    ...
    -----> Building runtime environment
    -----> Building static web assets with lineman
    Running "configure" task

    Running "common" task

    Running "coffee:compile" (coffee) task
    File generated/js/app.coffee.js created.
    File generated/js/spec.coffee.js created.
    >> Destination not written because compiled files were empty.

    Running "less:compile" (less) task
    File generated/css/app.less.css created.

    Running "jshint:files" (jshint) task
    >> 0 files lint free.

    Running "handlebars:compile" (handlebars) task
    >> Destination not written because compiled files were empty.

    Running "jst:compile" (jst) task
    File "generated/template/underscore.js" created.

    Running "configure" task

    Running "concat:js" (concat) task
    File "generated/js/app.js" created.

    Running "concat:spec" (concat) task
    File "generated/js/spec.js" created.

    Running "concat:css" (concat) task
    File "generated/css/app.css" created.

    Running "images:dev" (images) task
    Copying images to 'generated/img'

    Running "webfonts:dev" (webfonts) task
    Copying webfonts to 'generated/webfonts'

    Running "homepage:dev" (homepage) task
    Homepage HTML written to 'generated/index.html'

    Running "dist" task

    Running "uglify:js" (uglify) task
    File "dist/js/app.min.js" created.
    Uncompressed size: 38578 bytes.
    Compressed size: 4452 bytes gzipped (13217 bytes minified).

    Running "cssmin:compress" (cssmin) task
    File dist/css/app.min.css created.
    Uncompressed size: 69 bytes.
    Compressed size: 45 bytes gzipped (57 bytes minified).

    Running "images:dist" (images) task
    Copying images to 'dist/img'

    Running "webfonts:dist" (webfonts) task
    Copying webfonts to 'dist/webfonts'

    Running "homepage:dist" (homepage) task
    Homepage HTML written to 'dist/index.html'

    Done, without errors.
           Web assets built
    -----> Bundling Apache v2.2.19
    -----> Discovering process types
           Procfile declares types   -> (none)
           Default types for Lineman -> web

    -----> Compiled slug size: 34.8MB
    -----> Launching... done, v9
           http://pacific-fortress-3824.herokuapp.com deployed to Heroku

    To git@heroku.com:pacific-fortress-3824.git
       9b88fd7..e4bfa19  HEAD -> master

Apache Modules
-----

Custom Apache modules saved to a directory named `apache_modules` in your app root will be deployed to Heroku (under `apache/modules` in the slug). You'll need to make a corresponding change to `config/httpd.conf` – adding a line to load the module. Here's an example that will load the `headers_module` (assuming that `mod_headers.so` exists under `apache_modules` in your app root):

```
LoadModule headers_module  modules/mod_headers.so
```

__Building Apache Modules__

To build an Apache module, we'll use a Heroku one-off dyno:

```
$ heroku run bash
```

Once your one-off dyno is up and running, download a copy the Apache source, matching the version you'll be running in production. This buildpack uses 2.2.19.

```
$ curl http://archive.apache.org/dist/httpd/httpd-2.2.19.tar.gz | tar zx
```

Change to the Apache source directory:

```
$ cd httpd-2.2.19
```

Build the module with `apxs`

```
$ /app/apache/bin/apxs -i -c -Wl,lz modules/metadata/mod_headers.c
```

Last, copy (scp works great) the compiled module from the one-off dyno (`/app/apache/modules`) and commit it to your app's `apache_modules` directory (you might need to create this directory).


HTML5 PushState
-----

To add support for pushstate urls you can use the mod_rewrite module.  Follow the instructions above to build and add the mod_rewrite to your project.  The only difference to note when building is `mod_rewrite.c` is found in `modules/mappers` instead of `modules/metadata`:

```
$ /app/apache/bin/apxs -i -c -Wl,lz modules/mappers/mod_rewrite.c
```

Once you have mod_rewrite.so in your `apache_modules` directory you can add the following rewrite rule to `config/httpd.conf`:

```
# Handle push state urls
# any request that is NOT for a file or directory rewrite to index.html
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-f
    RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-d
    RewriteRule (.*) /index.html [L,QSA]
</IfModule>
```
