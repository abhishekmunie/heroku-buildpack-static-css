Heroku buildpack: Static-CSS
============================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpack) which compiles less files, minifies css and serves css files using static nginx.

Usage
-----

Example usage:

    $ ls -R *
    _staticcss.yml		       SassySCC.scss                   source.less                main.css                   lib.min.css
    ...

    $ heroku create --stack cedar --buildpack https://github.com/abhishekmunie/heroku-buildpack-static-css.git
    ...

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack... cloning with git...done
    -----> Static-CSS app detected
    -----> Resolving engine versions
    ...
    -----> Fetching Node.js binaries
    -----> Vendoring node into slug
    -----> Installing dependencies with npm
           less@1.3.0 ./node_modules/less
           less@1.3.0 /tmp/node.XHMNKL/node_modules/less
           Dependencies installed
    -----> Building runtime environment
    -----> Installing SASS...
           Successfully installed sass-3.2.1
           1 gem installed
           Installing ri documentation for sass-3.2.1...
           Installing RDoc documentation for sass-3.2.1...
           done.
    -----> Compiling SCSS...
           done.
    -----> Compiling LESS files...
           -----> compiling source.less...done
           ...
           done.
    -----> Fetching YUI Compressor 2.4.7...done
    -----> Compressing CSS files...
           -----> compressing source.css...done
           -----> compressing main.css...done
           ...
           done.
    -----> Creating default nginx configuration...done
    -----> Fetching nginx binaries
    -----> Vendoring nginx 1.0.14
    -----> Discovering process types
           Procfile declares types      -> (none)
           Default types for Static-CSS -> web
    ...

The buildpack will detect your app as Static CSS if it has the file `_staticcss.yml` in the `root`. At present `_staticcss.yml` doesn't support any configuration.
`.less` and `.scss` files will be removed after compilation.
You can set custom nginx config as described for [heroku-buildpack-nginx](https://github.com/abhishekmunie/heroku-buildpack-nginx).

Hacking
-------

To modify this buildpack, fork it on Github. Push up changes to your fork, then
create a test app with `--buildpack <your-github-url>` and push to it.

This buildpack first uses [LESS node.js command-line binary](http://lesscss.org/#-server-side-usage) to compile `filename.less` to `filename.scss`
then [SASS Ruby gem](http://sass-lang.com) to compile `.scss` & `.sass` files.
It simply runs `lessc "filename.less" > "filename.css"` and `sass --update $BUILD_DIR:$BUILD_DIR` on all `.less` files.
If the app has `config.rb` in `root`, it will be compiled using [Compass](http://compass-style.org/) inplace of standard scss compilation.
It then uses [YUI Compressor](https://yuilibrary.com/projects/yuicompressor/) to minify `filename.css` and create `filename.min.css`.
It also creates a copy of `filename.min.css` file with first 8 characters of its sha1 (`filename.<sha1:0:8>.css`).
It simply runs `java -jar yuicompressor-2.4.7.jar --type css -o "filename.min.css" "filename.css"` on all `.css` files except those ending in `.min.css`.