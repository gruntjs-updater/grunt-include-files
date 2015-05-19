# grunt-include-files

> Include your files into your HTML files automatically.

## Getting Started
This plugin requires Grunt `~0.4.4`

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```shell
npm install --save-dev grunt-include-files
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-include-files');
```

## The "includeFiles" task

### Overview
In your project's Gruntfile, add a section named `includeFiles` to the data object passed into `grunt.initConfig()`.

```js
grunt.initConfig({
  includeFiles: {
    options: {
      // Task-specific options go here.
    },
    your_target: {
      // Target-specific file lists and/or options go here.
    },
  },
})
```

### Options

#### options.basePath
Type: `String`
Default value: `''`

The base path to use when expanding files.

#### options.baseUrl
Type: `String`
Default value: `''`

The base URL to use for included files in the final result.
For example, setting `baseUrl` to `public/` will result in files being included from `public/path/to/your/file`.

#### options.templates
Type: `Object`

The templates for sources included in `html`, `haml`, `jade`, `scss`, `less` files. Definition of single template overrides its default equivalent only.

Example:
```js
includeFiles: {
  options: {
    basePath: 'app',
    baseUrl: 'public/',
    templates: {
      html: {
        js: '<script src="{filePath}"></script>',
        css: '<link rel="stylesheet" type="text/css" href="{filePath}" />',
        toc: '<a href="{filePath}">{fileName}</a>'
      },
      haml: {
        js: '%script{src: "{filePath}"}/',
        css: '%link{href: "{filePath}", rel: "stylesheet"}/'
      },      
      jade: {
        js: 'script(src="{filePath}", type="text/javascript")',    
        css: 'link(href="{filePath}", rel="stylesheet", type="text/css")'
      },
      scss: {
        scss: '@import "{filePath}";',
        css: '@import "{filePath}";',
      },
      less: {
        less: '@import "{filePath}";',
        css: '@import "{filePath}";',
      }
    }
  },
  myTarget: {
    files: {
      'dist/index.html': 'app/index.tpl.html'
    }
  }
}
```

As it was mentioned above, it is possible to override only necessary templates. 

#### options.rename
Type: `function`

A way of returning a custom filepath.

See [grunt.file.expandMapping](http://gruntjs.com/api/grunt.file#grunt.file.expandmapping).
If specified, this function will be responsible for returning the final dest filepath. By default, it joins dest and matchedSrcPath like so:
```js
  rename: function(dest, matchedSrcPath, options) {
    return path.join(dest, matchedSrcPath);
  }
 ```

#### options.flatten
Type: `Boolean`
Default value: `false`

Remove the path component from all matched src files. The src file path is still joined to the specified dest.

[grunt.file.expandMapping.flatten](http://gruntjs.com/api/grunt.file#grunt.file.expandmapping).

### Include syntax
Currently supported: `html`, `haml`, `jade`, `scss` and `less`

#### HTML syntax
`<!-- include: options_go_here_as_json -->`

#### SCSS and LESS syntax
`// include: options_go_here_as_json`

#### include.type
The type of files that are being included.
Necessary for choosing the template for output. See `options.templates` for currently supported types.
Supported types are dependend on the file type you include them from.

#### include.bower
**DEPRECATED**: Use another Grunt plugin which is better suited for this usecase, like [grunt-bower-install](https://github.com/stephenplusplus/grunt-bower-install).
This plugin doesn't support the `main` property for example and is mainly used for development purposes to directly include external sources.

Include files of the specified Bower component. The component should have a `bower.json` meta data with a property called `sources`.
This property should contain the files grouped by type, which are passed through the `grunt.file.expand` method.

Example:
```js
{
  "name": "package.name",
  ...
  "sources": {
    "js": [
      "src/scripts/**/*.js",
      "lib/compiled-templates.js"
    ],
    "css": "src/styles/**/*.css"
  }
}
```

#### include.files
Include the given files. Files are passed through the `grunt.file.expand` method.

#### include.basePath
Set to override the `basePath` set in the options.

#### include.baseUrl
Set to override the `baseUrl` set in the options.

#### include.ordering
Type: `String`
Default value: `undefined`

Ordering method to be used when including files. Currently supported methods are:
* `undefined` (default) - included files are sorted by their paths alphabetically in ascending order.
* `'top-down'` - files from the parent directory will be included before files from subdirectories.

#### Overwriting files

To set the source file as the destination file use an `/include` comment:

```html
<!-- include: "type": "js", "files": "js/**/*.js" -->
<script type="text/javascript" src="js/_first.js"></script>
<script type="text/javascript" src="js/lib/dep1.js"></script>
<!-- /include -->
```
  
When includeFiles is run it will keep the include comments and only update the includes inside it.

This works the same way for Less/Sass, by using `// /include`.

### Usage Examples
Configure your task like this:

```js
grunt.initConfig({
  includeFiles: {
    options: {
      basePath: 'app',
      baseUrl: 'public/'
    },
    myTarget: {
      files: {
        'dist/index.html': 'app/index.tpl.html'
      }
    }
  }
})
```

The file `index.tpl.html` could contain, for example:
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>Index</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width">

    <!-- 
      Automatically include Bower components. Use the "sources" object in your bower.json
      to specify which source files are which.
    -->
    <!-- include: "type": "css", "bower": "yourComponent" -->

    <!--
      Include CSS files from a "tmp" directory, put there by another task.
      This shows how to override the default "basePath" set in the options.
    -->
    <!-- include: "type": "css", "basePath": "tmp", "files": "styles/**/*.css" -->
  </head>
  <body>
    <!-- include: "type": "js", "bower": "yourComponent" -->
    <!-- include: "type": "js", "files": "scripts/**/*.js" -->
  </body>
</html>
```
And the resulting file `index.html` will look something like:
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>Index</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width">

    <!-- 
      Automatically include Bower components. Use the "sources" object in your bower.json
      to specify which source files are which.
    -->
    <!-- include: "type": "css", "bower": "yourComponent" -->
        
    <!--
      Include CSS files from a "tmp" directory, put there by another task.
      This shows how to override the default "basePath" set in the options.
    -->
    <link href="public/styles/main.css" rel="stylesheet" type="text/css" />
    <link href="public/styles/anotherFile.css" rel="stylesheet" type="text/css" />
  </head>
  <body>
    <script src="public/bower_components/yourComponent/main.js"></script>
    <script src="public/scripts/app.js"></script>
    <script src="public/scripts/anotherFile.js"></script>
    <script src="public/scripts/controllers/evenMore.js"></script>
  </body>
</html>
```

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint and test your code using [Grunt](http://gruntjs.com/).

## Release History

### 2015-05-19 ver. 0.5.3

* added sass(indent syntax) for including target.

### 2015-05-19 ver. 0.5.2

* added function to get baseName (filename without extention).

### 2015-05-19 ver. 0.5.1

* changed plugin name to `grunt-include-files`

### 2015-05-19 ver. 0.5.0

* add function to use file name. This function will be useful for especially creating toc pages.

### 2014-11-22 ver. 0.4.4

* improve endline detected when scanning for indentation

### 2014-11-22 ver. 0.4.3

* replace `grunt.template.process` with `grunt.config.process` for better support (see https://github.com/jwvdiermen/grunt-include-source/pull/27)

### 2014-10-26 ver. 0.4.2

* bug fix: line endings are not detected when concatenating multiple source files.
* remove redundant "type" attribute in `<script>` tag
* add support for templates in file pattern

### 2014-07-12 ver. 0.4.1

* bug fix: multiple lines are detected as the separator, but is should only look at the first line.

### 2014-07-12 ver. 0.4.0

* add support to overwrite the source file by including an end tag
* detect EOL globally from file when no indentation can be found (e.a. when at the begin of a file)

### 2014-05-04 ver. 0.3.9

* add support for Jade
* make automatic indentation more intelligent by using EOL from context

### 2014-05-03 ver. 0.3.8

* replace all instances of {filePath} instead of the only the first
* add {filePathDecoded} to enable use of raw URL which isn't encoded

### 2014-04-27 ver. 0.3.7

* DEPRECATED: Bower support, see readme.
* support for HAML (see https://github.com/jwvdiermen/grunt-include-source/pull/17)
* tags generated for HTML now use HTML5 syntax by default.
* added basic tests
* updated dependencies.

### 2014-04-18 ver. 0.3.6

* added support for `include.ordering` option. 
* added automated indentation for included entries (see https://github.com/jwvdiermen/grunt-include-source/issues/14)
