---
layout: post
title: Getting project bundling quickly to start Reactjs development
categories: [blog, howto, frontend]
tags: [tutorial, frontend, beginner]
comments: true
#series: "Cloud lab with OpenStack"
---

I'm a sporadic front-end developer and have to learn virtually from the beginning every time I need a web front-end. This is because I forget a lot and also because of the rapid development of front-end technologies and tools. The first pass to cross is setting up the project and bundling tools. Unfortunately plenty of tutorials on the web are also obsolete. I will try to keep this post evolving but do check the date!. If you are in a similar situation, this article can hopefully help. Web project seems to be best managed by gulpjs, webpack or rollup **bundler** which can be extended with various plugins to meet most customization to the desired front-end distribution (js+css). From my own really quick survey, _gulpjs_ seem old, _rollup_ is newer and quick, _webpack_ is most customizable but most complex. However, a combination of gulps and rollup can get me started quickly with following requirements:

* _Reactjs_.
* Web server with _browsersync_ (already there from cookiescutter-django template!).
* Crawl imported js and css library and bundling them single js and css in the distribution. Easier to setup Content Security Policy rules.

Other added or useful features coming from the selected tools:

* Future js, css [syntax](https://css-tricks.com/bridging-the-gap-between-css-and-javascript-css-modules-postcss-and-the-future-of-css/).
* A lot other things.

## TL;DR

Clone this [repo](https://github.com/thuydang/gulp_rollup_react) and start coding Reactjs. Because of the _postcss-modules_ plugin, `Classname` attributes are declared differently.

Instead of:
    import React from "react";
    import style from "./panel.css";
    const Panel = () => {
      return (
          <div className="panelDefault">
          <div className="panelBody">A Basic Panel</div>
          </div>
          )
    }
    export default Panel; 

Use:

    <!-- container is a class from a globle css (import from index.html) -->
    <div className={`${style.panel} ${style.panelDefault} container`}>
        <div className={style[panel-body]>A Basic Panel</div>
    </div>

, or multiple classes with _classname_ plugin:

    @import cn from "classnames";
    <div className={cn(style.panel,style.panelDefault,container)}>
    </div>

    
This is annoying for me too because I want to just copy html code without having to change css class name declaration!. If I find a better way, this will disappear from the post!. If you get over it for now run the bundling and server from the project's folder and see your front-end:

    npm install
    gulp


##  How things are set up and what for

If you want to improve the bundling or customize the project, please read on. We will look at the code and explanations. All the required packages are in _package.json_ as many year ago, also now configuration is done there. All the plugin configurations are in _gulpfile.js_ and gulpjs is the overall bundler. I don't want to use dot files for some of the plugins although it is possible.  

### Gulpjs

Gulpjs allow us to define _tasks_, which are executed with the `gulp task-name` command or without task name for default task. We have tasks for bundling project artefacts and tasks for running the web server.

#### Declaring plugins

At the beginning of the _gulpfile.js_ the plugins are imported as below: 

    // rollup plugins
    const rollup = require('rollup');
    const babel = require('@rollup/plugin-babel').default
    const rollupResolve = require('@rollup/plugin-node-resolve').default
    const commonJs = require('@rollup/plugin-commonjs')
    const postcss = require('rollup-plugin-postcss')
    const postcssModules = require('postcss-modules')
    const image = require('@rollup/plugin-image')
    const replace = require('@rollup/plugin-replace')

They are installed with:
    npm i -SD @rollup/plugin-replace @rollup/plugin-image

Or to install everything in _package.json_:
    npm install

#### Rollup configurations

    // rollup
    function build_dev(){
      return rollup.rollup({
        input: `${paths.src}/index.js`,
        plugins: [
          // Set env to compile React for dev or prod. Required by React.
          replace({
            //'process.env.NODE_ENV': JSON.stringify( 'production' )
            'process.env.NODE_ENV': JSON.stringify( 'development' )
          }),
          babel(
            {
              presets: [
                "@babel/preset-env", 
                "@babel/preset-react"
              ],
              exclude: ['node_modules/**', '*.css', /.css$/],
              ignore: ['*.css', /.(css|json|svg)$/],
              babelHelpers: 'bundled',
              plugins: [
    
              ],
              babelrc: false
            }
          ),
          //peerDepsExternal(),
          postcss({
            plugins: [
              // https://medium.com/grandata-engineering/how-i-set-up-a-react-component-library-with-rollup-be6ccb700333
              postcssModules({
                getJSON (id, exportTokens) {
                  cssExportMap[id] = exportTokens;
                }
              })
            ],
            getExportNamed: false,
            getExport (id) {
              return cssExportMap[id];
            },
            // Save it to a .css file
            extract: 'bundle.css', // to be placed in output folder. Can also set to true,
            sourceMap: true,
            modules: true,
            //use: ['sass'],
          }),
          image(),
          rollupResolve(),
          commonJs()
        ]
      }).then(function(bundle) {
        return bundle.write({
          file: `${paths.build}/bundle.js`,
          format: 'iife',
          //format: 'umd', // for bundling lib
          //name: 'library', // for bundling lib
          name: 'app',
          sourcemap: true
        })
      }).catch(err => console.log(err))
    }
    
#### Browser sync server
    // Browser sync server for live reload
    function initBrowserSync() {
        browserSync.init(
          [
            `${paths.css}/*.css`,
            `${paths.js}/*.js`,
            `${paths.public}/*.html`
          ], {
            // https://www.browsersync.io/docs/options/#option-server
            server: {
              baseDir: `${paths.public}`,
              index: "index.html",
              middleware: function (req, res, next) {
                res.setHeader("Content-Security-Policy", "base-uri 'self'; connect-src 'self' 'unsafe-inline'; font-src 'self' data: 'unsafe-inline'; frame-ancestors 'self' 'unsafe-inline'; frame-src 'self' 'unsafe-inline'; img-src 'self' 'unsafe-inline' data:; manifest-src 'self' 'unsafe-inline'; media-src 'self' 'unsafe-inline' data:; object-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'");
                res.setHeader("X-Content-Security-Policy", "base-uri 'self'; connect-src 'self'; font-src 'self' data:; frame-ancestors 'self'; frame-src 'self'; img-src 'self'; manifest-src 'self'; media-src 'self'; object-src 'self'; script-src 'self'; style-src 'self'");
                next();
              }
            },
            port: 4000,
            ui: {
              port: 4001
            },
            // https://www.browsersync.io/docs/options/#option-open
            // Disable as it doesn't work from inside a container
            open: false
          }
        )
    }
    
#### Auto rebuilt bundle on code changed
    // Watch
    function watchPaths() {
      //watch(`${paths.sass}/*.scss`, styles)
      //watch(`${paths.css}/*.css`, styles)
      watch(`${paths.public}/**/*.html`).on("change", reload)
      watch([`${paths.js}/*.js`, `${paths.css}/*.css`, `!${paths.js}/*.min.js`], build_dev).on("change", reload)
    }
    
#### Defining task execution sequence
    // Generate all assets
    const generateAssets = parallel(
      build_dev,
      imgCompression
    )
    
    // Set up dev environment
    const dev = parallel(
      initBrowserSync,
      watchPaths
    )
    
    exports.default = series(generateAssets, dev)
    exports["generate-assets"] = generateAssets
    exports["dev"] = dev


## References to useful articles

* <https://css-tricks.com/bridging-the-gap-between-css-and-javascript-css-modules-postcss-and-the-future-of-css/>
* <https://medium.com/grandata-engineering/how-i-set-up-a-react-component-library-with-rollup-be6ccb700333>
* <https://starbeamrainbowlabs.com/blog/article.php?article=posts/337-20190121-Rollup.html>
* <https://blog.yipl.com.np/css-modules-with-react-the-complete-guide-a98737f79c7c>
