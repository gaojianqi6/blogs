---
title: React和Webpack项目构建
date: 2016-08-25 18:24:07
tags: [javascript, React, Webapck]
categories: [技术]
---
这次在项目中使用了React，并使用webpack来打包js文件，gulp来编译stylus等其他事情。

最开始我们想达到的效果，希望能够根据webpack能够把reactjs打包成项目可以使用的文件，在项目目录中新建webpack.config.js文件

``` javascript
var path = require('path');
var webpack = require('webpack');

var BUILD_DIR = path.resolve(__dirname, 'content/jsx');
var APP_DIR = path.resolve(__dirname, 'content/scripts');

module.exports = {
	//是否让webpack监听入口相关的文件
	watch: true,
	//配置入口文件，入口文件可以以对象的形式配置，也可以以数组的形式配置,后缀名可以省略
    entry: {
        index: path.join(BUILD_DIR, 'index.jsx')
    },
    //输出文件出口
    output: {
    	/*
         输出路径，在这我们要手动创建一个文件夹，名字可以自己命名，
         输出的文件路径是相对于本文件的路径
        **/
        path: APP_DIR,    //输出路径
        filename: '[name].bundle.js'   //输出文件名，文件可以自己定义，[name]的意思是与入口文件的文件对应，可以不用[name]
    },
    resolve: {
        extensions: ['', '.js', '.jsx'],
        /*
        * 别名配置，配置之后，可以在别的js文件中直接使用require('d3')，将导入的文件作为一个模块导入到你需要的项目中，不用配置别也可会当作模块导入项目中，只是你要重复写路径而已。
        **/
        alias: {
            'd3': 'd3/d3.min.js'
        }
    },
    module: {
    	/*
	    * 标题：加载器（loaders）
	    * 作用：需要下载不同别的加载器，如css，js，png等等
	    **/
        loaders: [
            {
                test: /\.js(x?)$/,
                exclude: /(node_modules|bower_components)/,
                loader: 'babel',
                query: {
                    presets: ['react', 'es2015', 'stage-0']
                }
            }
        ],
    }
};

```
Webpack的一些命令：

``` bash
$ webpack --config XXX.js //使用另一份配置文件（比如webpack.config2.js）来打包
$ webpack --watch //监听变动并自动打包
$ webpack -p//压缩混淆脚本，这个非常非常重要！
$ webpack -d//生成map映射文件，告知哪些模块被最终打包到哪里了
```

但是这样打包出来的文件会很大，

这个可以使用webpack的externals来设置webpack将依赖的库指向全局变量，从而不导入这个js文件。

但是如果引用的包比较多的情况下，维护起来会比较麻烦，所以使用webpack的dll plugin来解决这个问题。
使用这个功能需要把打包过程分成两步：

1. 打包ddl包
2. 引用ddl包，打包业务代码

首先建一个，webpack.dll.config.js文件

``` javascript
var webpack = require('webpack');
var path = require('path');

var APP_DIR = path.resolve(__dirname, 'content/scripts');

var vendors = [
    "react",
    "react-bootstrap",
    "react-dom",
    "react-redux",
    "react-router",
    "react-router-redux",
    "redux",
    "redux-logger",
    "redux-promise",
    "redux-thunk",
    "whatwg-fetch"
];

module.exports = {
    output: {
        path: path.join(APP_DIR, '/'),
        filename: '[name].dll.js',
        /**
	     * output.library
	     * 将会定义为 window.${output.library}
	     * 在这次的例子中，将会定义为`window.[name]_[chunkhash]`
	    */
        library: '[name]_[chunkhash]',
    },
    entry: {
        vendor: vendors,
    },
    plugins: [
        new webpack.DllPlugin({
        	/**
		    * path
		    * 定义 manifest 文件生成的位置
		    * 也可以设置[name]manifest.json,[name]部分由entry的名字替换
		    */
            path: 'manifest.json',
            /**
		    * name
		    * dll bundle 输出到那个全局变量上
		    * 和 output.library 一样即可。 
		    */
            name: '[name]_[chunkhash]',
            context: __dirname
        })
    ]
};
```
webpack.DllPlugin 的选项中：

path 是 manifest.json 文件的输出路径，这个文件会用于后续的业务代码打包；
name 是 dll 暴露的对象名，要跟 output.library 保持一致；
context 是解析包路径的上下文，这个要跟接下来配置的 webpack.config.js 一致。

PS：自己用gulp来执行的webpack打包，只打出来vendor文件，没有出来manifest.json，改用webpack命令打包就好了。


运行之后会生成两个文件，一个是所有引用的js文件，另一个是manifest.json:
``` bash
➜  webapp git:(dev_jerome) ✗ ./node_modules/.bin/webpack --config webpack.dll.config.js 
Hash: fd919a411af8bb56f919
Version: webpack 1.13.2
Time: 2389ms
        Asset    Size  Chunks             Chunk Names
vendor.dll.js  1.6 MB       0  [emitted]  vendor
   [0] dll vendor 12 bytes {0} [built]
    + 533 hidden modules
➜  webapp git:(dev_jerome) ✗ 

```

再在原来的webpack.config文件里面增加DllReferencePlugin的配置，需要注意的是这里面的context需要和dll.config里面的context一样。

```javascript
var path = require('path');
var webpack = require('webpack');

var BUILD_DIR = path.resolve(__dirname, 'content/jsx');
var APP_DIR = path.resolve(__dirname, 'content/scripts');

module.exports = {
    watch: true,
    entry: {
        index: path.join(BUILD_DIR, 'index.jsx')
    },
    output: {
        path: APP_DIR,
        filename: '[name].bundle.js'
    },
    resolve: {
        extensions: ['', '.js', '.jsx']
    },
    module: {
        loaders: [
            {
                test: /\.js(x?)$/,
                exclude: /(node_modules|bower_components)/,
                loader: 'babel',
                query: {
                    presets: ['react', 'es2015', 'stage-0']
                }
            }
        ],
    },
    externals: {
        "jquery": "jQuery"
    },
    plugins: [
        new webpack.DllReferencePlugin({
            context: __dirname,
            manifest: require('./manifest.json')
        }),
        new webpack.NoErrorsPlugin()
    ]
};

```
webpack.DllReferencePlugin 的选项中：

context 需要跟之前保持一致，这个用来指导 Webpack 匹配 manifest.json 中库的路径；
manifest 用来引入刚才输出的 manifest.json 文件。
DllPlugin 本质上和我们手动分离第三方库是一样的，但是对依赖包极多的项目来说，自动化明显加快了生产效率。



可以在package.json里面设置运行脚本,如果依赖的脚本没有变化的话可以只用执行bundle任务。
``` javascript
"scripts": {
    "build": "webpack --config webpack.dll.config.js && gulp bundle",
    "bundle": "./node_modules/.bin/gulp bundle"
}
```

下面是依赖的gulp文件

``` javascript
var gulp = require('gulp');
var webpack = require('webpack');
var webpackStream = require('webpack-stream');
var webpackConfig = require('./webpack.config.js');
var path = require('path');
var watch = require('gulp-watch');
var gulpWebpack = require('gulp-webpack');
var gutil = require('gulp-util');

gulp.task('bundle', function () {
    var ENTYR_DIR = path.join(__dirname, "content/jsx");
    var DIST_DIR = path.join(__dirname, "content/scripts");

    return gulp.src(path.join(ENTYR_DIR, "index.jsx"))
        .pipe(webpackStream(require('./webpack.config.js')))
        .pipe(gulp.dest(DIST_DIR));
});

gulp.task('bundle-dll', function () {

    var myConfig = Object.create(require('./webpack.dll.config.js'));
    myConfig.plugins = [
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.UglifyJsPlugin()
    ];

    // run webpack
    webpack(myConfig, function(err, stats) {
        if (err) throw new gutil.PluginError('webpack', err);
        gutil.log('[webpack]', stats.toString({
            colors: true,
            progress: true
        }));
        callback();
    });
});



gulp.task('bundle-watch', function () {

    var ENTYR_DIR = path.join(__dirname, "content/jsx");
    var DIST_DIR = path.join(__dirname, "content/scripts");

    return gulp.src(['content/**/*.jsx', 'content/**/*.js'])
        .pipe(watch(function () {
            return gulp.src(path.join(ENTYR_DIR, "index.jsx"))
                .pipe(webpackStream(webpackConfig))
                .pipe(gulp.dest(DIST_DIR));
        }));
});

gulp.task('webpack', function(callback) {
    var myConfig = Object.create(webpackConfig);
    myConfig.plugins = [
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.UglifyJsPlugin()
    ];

    // run webpack
    webpack(myConfig, function(err, stats) {
        if (err) throw new gutil.PluginError('webpack', err);
        gutil.log('[webpack]', stats.toString({
            colors: true,
            progress: true
        }));
        callback();
    });
});

gulp.task('gulpWebpack', function() {
    var ENTYR_DIR = path.join(__dirname, "content/jsx");
    var DIST_DIR = path.join(__dirname, "content/scripts");

    return gulp.src(path.join(ENTYR_DIR, "index.jsx"))
        .pipe(gulpWebpack(webpackConfig))
        .pipe(gulp.dest(DIST_DIR));
});

```

参考文章: [如何十倍提高你的webpack构建效率](http://blog.csdn.net/u011413061/article/details/51872412)


































