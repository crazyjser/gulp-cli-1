# 前端入门之前端自动化工具（Gulp）
前端自动化之gulp最佳实践，快速构建自己的自动化工具，是学习gulp最好指南!

## 1、Gulp概述
将重复工作抽象成一个个的任务，机械化处理这些重复的任务。
## 2、Gulp四大接口

### 2.1获取文件源gulp.src();
### 2.2转移文件到目标gulp.dest();
### 2.3监听文件变化gulp.watch();
### 2.4添加执行任务gulp.task();

## 3、Gulp常用插件

[gulp插件黑名单](https://github.com/gulpjs/plugins/blob/master/src/blackList.json)

### 3.1公共插件

- [自动加载各种插件,免require（）加载，调用时直接省略gulp-：gulp-load-plugins,作用：静态文件服务器，同时也支持浏览器自动刷新](https://www.npmjs.com/package/gulp-load-plugins)

``` js
var gulpLoadPlugins = require('gulp-load-plugins');
var pg = gulpLoadPlugins();
pg.jshint //插件帮我自动加载require('gulp-jshint');
pg.concat //插件帮我自动加载require('gulp-concat');
```

- [数据流的控制：pump](https://www.npmjs.com/package/pump)
- [重命名文件：gulp-rename](https://www.npmjs.com/package/gulp-rename)

``` js
// rename via hash 
gulp.src("./src/main/text/hello.txt", { base: process.cwd() })
.pipe(rename({
	dirname: "main/text/ciao",
	basename: "aloha",
	prefix: "bonjour-",
	suffix: "-hola",
	extname: ".md"
}))
.pipe(gulp.dest("./dist")); 
// ./dist/main/text/ciao/bonjour-aloha-hola.md 
```
- [映射源文件：gulp-sourcemaps](https://www.npmjs.com/package/gulp-sourcemaps)
- [合并css或js文件：gulp-concat](https://www.npmjs.com/package/gulp-concat)

``` js
gulp.task('javascript', function() {
  return gulp.src('src/**/*.js')
    .pipe(sourcemaps.init())
      .pipe(concat('all.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('dist'));
});
```

- [html版本添加：gulp-usemin](https://www.npmjs.com/package/gulp-usemin)
- [js&css文件自动增加md5后缀：gulp-dev](https://www.npmjs.com/package/gulp-dev)


``` html
<!-- build:<pipelineId>(alternate search path) <path> -->
	pipelineId:html/css/js或inlinecss、inlinejs、remove
	path:新文件路径或文件名
<!-- endbuild -->
例如：
<!-- build:css style.css -->
<link rel="stylesheet" href="css/clear.css"/>
<link rel="stylesheet" href="css/main.css"/>
<!-- endbuild -->
```
``` js

gulp.task('usemin', function() {
  return gulp.src('./*.html')
    .pipe(usemin({
      css: [ rev ],
      html: [ function () {return htmlmin({ collapseWhitespace: true });} ],
      js: [ uglify, rev ],
      inlinejs: [ uglify ],
      inlinecss: [ cleanCss, 'concat']
    }))
    .pipe(gulp.dest('build/'));
});
```

- [清空文件：使用nodejs模块（del vinyl-paths）实现，gulp插件废弃了](https://github.com/gulpjs/gulp/blob/master/docs/recipes/delete-files-folder.md)

``` js
var del = require('del');
var vinylPaths = require('vinyl-paths');

gulp.task('clean:tmp', function () {
  return gulp.src('tmp/*')
    .pipe(vinylPaths(del))
    .pipe(stripDebug())
    .pipe(gulp.dest('dist'));
});
```
- [多浏览器静态服务：browser-sync](https://www.npmjs.com/package/browser-sync)
``` js
var browserSync = require('browser-sync').create();
gulp.task('serve', function () {

    // Serve files from the root of this project
    browserSync.init({
    	files: "**",
        port:"1688",
        server: {
            baseDir: "./src",
        }
    });

    gulp.watch("dist/**").on("change", browserSync.reload);
});
```
### 3.2处理html文件插件

- [压缩html文件 gulp-htmlmin](https://www.npmjs.com/package/gulp-htmlmin)
``` js
gulp.task('html', function() {
  return gulp.src('src/*.html')
    .pipe(htmlmin({collapseWhitespace: true}))
    .pipe(gulp.dest('dist'));
});
```
### 3.3处理css文件插件
- [编译 sass：gulp-sass](https://www.npmjs.com/package/gulp-sass)
- [编译 Less：gulp-less](https://www.npmjs.com/package/gulp-less)
``` js
gulp.task('sass', function () {
  return gulp.src('./sass/**/*.scss')
    .pipe(sass.sync().on('error', sass.logError))
    .pipe(gulp.dest('./css'));
});
gulp.src('./less/**/*.less')
  .pipe(sourcemaps.init())
  .pipe(less())
  .pipe(sourcemaps.write())
  .pipe(gulp.dest('./public/css'));
```
- [最小化 css 文件：gulp-clean-css](https://www.npmjs.com/package/gulp-clean-css)

``` js
gulp.task('minify-css', function() {
    return gulp.src('./src/*.css')
        .pipe(sourcemaps.init())
        .pipe(cleanCSS())
        .pipe(sourcemaps.write())
        .pipe(gulp.dest('dist'));
});

```
- [自动添加前缀：gulp-autoprefixer](https://www.npmjs.com/package/gulp-autoprefixer)
``` js
gulp.task('default', () =>
    gulp.src('src/**/*.css')
        .pipe(sourcemaps.init())
        .pipe(autoprefixer())
        .pipe(concat('all.css'))
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('dist'))
);
```

### 3.4处理js文件插件

- [gulp-babel转前提：babel-preset-es2015](https://www.npmjs.com/package/babel-preset-es2015)
- [es2016格式转es2015：gulp-babel](https://www.npmjs.com/package/gulp-babel)
``` js
gulp.task('default', () => {
    return gulp.src('src/**/*.js')
        .pipe(sourcemaps.init())
        .pipe(babel({
            presets: ['es2015']
        }))
        .pipe(concat('all.js'))
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('dist'));
});
```
- [最小化 js 文件：gulp-uglify](https://www.npmjs.com/package/gulp-uglify)
``` js
gulp.task('uglify', function (cb) {
  pump([
        gulp.src('lib/*.js'),
        uglify(),
        gulp.dest('dist')
    ],
    cb
  );
});
```
- [校验 js 文件：gulp-jshint](https://www.npmjs.com/package/gulp-jshint)
``` js
gulp.task('lint', function() {
  return gulp.src('./lib/*.js')
    .pipe(jshint())
    .pipe(jshint.reporter('YOUR_REPORTER_HERE'));
});
```

### 3.5处理images图片文件插件

- [最小化图像：gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin)
- [最小化图像：imagemin-pngquant](https://www.npmjs.com/package/imagemin-pngquant)
``` js
gulp.task('default', () =>
    gulp.src('src/images/*')
        .pipe(imagemin([
		    imagemin.gifsicle({interlaced: true}),
		    imagemin.jpegtran({progressive: true}),
		    imagemin.optipng({optimizationLevel: 5}),
		    imagemin.svgo({plugins: [{removeViewBox: true}]})
		]))
        .pipe(gulp.dest('dist/images'))
);
//gifsicle — Compress GIF images
//jpegtran — Compress JPEG images
//optipng — Compress PNG images
//svgo — Compress SVG images
```

## 4 安装gulp配置文件

### 4.1安装 gulp 命令行工具
```shell
npm init
npm install -g gulp
```
### 4.2 安装插件

```shell
npm install gulp gulp-load-plugins gulp-minify-html gulp-usemin gulp-dev gulp-less gulp-clean-css gulp-autoprefixer gulp-uglify gulp-babel babel-preset-es2015 gulp-jshint gulp-imagemin gulp-rename gulp-concat gulp-clean gulp-sourcemaps browser-sync -D
```
### 4.2.1 `package.json`文件开发依赖如下：

``` js
"devDependencies": {
    "babel-preset-es2015": "^6.24.1",
    "browser-sync": "^2.18.11",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^4.0.0",
    "gulp-babel": "^6.1.2",
    "gulp-clean": "^0.3.2",
    "gulp-clean-css": "^3.3.1",
    "gulp-concat": "^2.6.1",
    "gulp-dev": "^0.3.0",
    "gulp-imagemin": "^3.2.0",
    "gulp-jshint": "^2.0.4",
    "gulp-less": "^3.3.0",
    "gulp-load-plugins": "^1.5.0",
    "gulp-minify-html": "^1.0.6",
    "gulp-rename": "^1.2.2",
    "gulp-sourcemaps": "^2.6.0",
    "gulp-uglify": "^2.1.2",
    "gulp-usemin": "^0.3.28",
    "jshint": "^2.9.4"
  }
```
### 4.3创建Gulp配置文件`gulpfile.js`







