# diy [![NPM version](https://badge.fury.io/js/diy.svg)](http://badge.fury.io/js/diy)

> do it yourself - build DSL

# install

```shell
npm install diy-build
```

# example
## concatenate and minify javascript files

Assume you have two files (`src/file1.js` and `src/file2.js`) that you want to
concatenate and minify into a single `_site/client.js` file. Here's how you'd
write a Diy (ES6) program for this task:

## diy source file

```js
/* configure.js */

var {
  generateProject
} = require("diy");

generateProject(_ => {
  _.collect("all", _ => {
    _.toFile( "_site/client.js", _ => {
      _.minify( _ => {
        _.concat( _ => {
          _.copy("src/*.js")
        })
      })
    })
  })
})
```

## description

Everything starts with the `generateProject` library function. You invoke it with
a closure that describes the final makefile targets (using `_.collect`). A target is just a name
you give to a set of products you want to build.

`_.toFile` creates a named file whose contents are created as described by the closure passed as second parameter.
The closure (as it can be intuitively understood) minifies the concatenation of a copy of javascript files in `src/*.js`.



To generate the makefile we use babel to get ES5:

```bash
babel configure.js | node
```


The makefile comes with two default targets (`prepare` and `clean`) plus all the targets defined with `collect`:

```bash
> make prepare      # Creates destination directories
> make clean        # Removes all products
> make all          # Execute commands associated with `all`
```

Make provides a way to specify the maximum parallelism to be used for building targets:

```bash
> make all -j 8     # Build all, execute up to 8 concurrent commands.
```



# customization

What about your favorite css/js preprocessor and other minifiers?

Here's how you would define a new processing step to compile javascript with a
bunch of browserify plugins:

```js
_.browserify = (src, ...deps) => {
  var command = (_) => `./node_modules/.bin/browserify -t liveify -t node-lessify  ${_.source} -o ${_.product}`
  var product = (_) => `${_.source.replace(/\..*/, '.bfd.js')}`
  _.compileFiles(...([ command, product, src ].concat(deps)))
}
```

`_.compileFiles` is a built in function to easily construct new processing steps. Its first
two parameters are two templates:

1. a function to build the command line
2. a function to build the product name

The remaining parameters are `src` (glob for the source files) and the source dependencies.

```js
generateProject(_ => {

  _.browserify = (dir, ...deps) => {
    var command = (_) => `./node_modules/.bin/browserify -t liveify -t node-lessify  ${_.source} -o ${_.product}`
    var product = (_) => `${_.source.replace(/\..*/, '.bfd.js')}`
    _.compileFiles(...([ command, product, dir ].concat(deps)))
  }

  _.collect("all", _ => {
    _.toFile( "_site/client.js", _ => {
        _.browserify("src/index.ls", "src/**/*.less", "src/**/*.ls")
    })
  })
}
```

# serving and livereloading

Serving static files from a directory and livereloading upon a change of a product is supported through `pm2` and `tiny-lr`. We can
create two make targets (`start` and `stop`) that take care of starting and stopping both services:

```js
generateProject(_ => {

    /* ... */

  _.collect("start", _ => {
    _.startWatch("_site/**/*")
    _.startServe("_site")
  })

  _.collect("stop", _ => {
    _.stopWatch()
    _.stopServe()
  })

    /* ... */
})
```

`_.startWatch(glob)` is a built-in step that launches a tiny-lr instance that triggers a reload upon change on files matching the glob.
`_.startServe(root,port)` serves files from the specified root and port.



## Author

**Vittorio Zaccaria**

+ [github/vzaccaria](https://github.com/vzaccaria)

## License
Copyright (c) 2015 Vittorio Zaccaria  
Released under the  license

***

_This file was generated by [verb-cli](https://github.com/assemble/verb-cli) on March 10, 2015._
