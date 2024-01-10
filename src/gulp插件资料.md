## 匹配符

```javascript
*： 匹配某个文件夹下面某一类型的文件，比如 gulp.src('.js/*.js'),js文件夹下面所有的js文件。
**：匹配某个文件夹下面0个或者多个文件夹，gulp.src('./js/**/*.js')，匹配js文件夹下面的子文件夹里面的js文件。
!: gulp.src('./js/*.js, !./js/index.js'),匹配js文件夹下面除了index.js文件的所有js文件。
{}: gulp.src('./js/**/{a,b}.js'), 匹配{}里面的文件。
```

## 文件操作

```javascript
del（替代gulp-clean）
    var del = require('del');
    del('./dist') // 删除整个dist文件夹
    
gulp-rename:重命名文件
    var rename = require('gulp-rename');
    gulp.src('./hello.txt')
    .pipe(rename('gb/goodbye.md'))
    .pipe(gulp.dest('./dist'))
    
    gulp.src('./hello.txt')
      .pipe(rename({
        dirname: "text",                // 路径名
        basename: "goodbye",            // 主文件名
        prefix: "pre-",                 // 前缀
        suffix: "-min",                 // 后缀
        extname: ".html"                // 扩展名
      }))
      .pipe(gulp.dest('./dist'));

gulp-concat:合并文件
    var concat = require('gulp-concat');

    gulp.src('./js/*.js')
        .pipe(concat('all.js'))         // 合并all.js文件
        .pipe(gulp.dest('./dist'));
        
    gulp.src(['./js/demo1.js','./js/demo2.js','./js/demo2.js'])
        .pipe(concat('all.js'))         // 按照[]里的顺序合并文件
        .pipe(gulp.dest('./dist'));


gulp-csso
描述：压缩优化css。
    
    var csso = require('gulp-csso');
    
    gulp.src('./css/*.css')
        .pipe(csso())
        .pipe(gulp.dest('./dist/css'))

gulp-html-minify
描述：压缩HTML。

    var htmlminify = require('gulp-html-minify');
    
    gulp.src('index.html')
        .pipe(htmlminify())
        .pipe(gulp.dest('./dist'))

gulp-imagemin
描述：压缩图片。

    var imagemin = require('gulp-imagemin');
    
    gulp.src('./img/*.{jpg,png,gif,ico}')
        .pipe(imagemin())
        .pipe(gulp.dest('./dist/img'))

gulp-zip
描述：ZIP压缩文件。

var zip = require('gulp-zip');

gulp.src('./src/*') // 如果需要打包多级目录可以使用[]
    .pipe(zip('all.zip'))                   // 压缩成all.zip文件
    .pipe(gulp.dest('./dist'))
```

## js/css自动注入

```javascript
gulp-useref
描述：在html中通过build提前确认之后打包完引入的是哪个文件。

// index.html

<!-- build:css /css/all.css -->
<link rel="stylesheet" href="css/normalize.css">
<link rel="stylesheet" href="css/main.css">
<!-- endbuild -->

// gulpfile.js

var useref = require('gulp-useref');

gulp.src('index.html')
    .pipe(useref())
    .pipe(gulp.dest('./dist'))
替换之后的index.html中就会变成：

<link rel="stylesheet" href="css/all.css">  // 之前的两个<link>替换成一个了

gulp-rev
描述：给静态资源文件名添加hash值:unicorn.css => unicorn-d41d8cd98f.css

var rev = require('gulp-rev');

gulp.src('./css/*.css')
    .pipe(rev())
    .pipe(gulp.dest('./dist/css'))
```

## 流控制

```javascript
gulp-if
描述：按某特定的判断运行不同的任务
var gulpif = require('gulp-if');
var uglify = require('gulp-uglify');
var concat = require('gulp-concat');
var condition = true; 

gulp.src('./js/*.js')
    .pipe(gulpif(condition, uglify(), concat('all.js')))  // condition为true时执行uglify(), else 执行concat('all.js')
    .pipe(gulp.dest('./dist/'));
```

## 工具

```javascript
gulp-sass
描述：编译sass。

var sass = require('gulp-sass');

gulp.src('./sass/**/*.scss')
    .pipe(sass({
        outputStyle: 'compressed'           // 配置输出方式,默认为nested
    }))
    .pipe(gulp.dest('./dist/css'));
    
gulp.watch('./sass/**/*.scss', ['sass']);   // 实时监听sass文件变动,执行sass任务

gulp-babel
描述：将ES6代码编译成ES5。

var babel = require('gulp-babel');

gulp.src('./js/index.js')
    .pipe(babel({
        presets: ['es2015']
    }))
    .pipe(gulp.dest('./dist'))


gulp-nodemon
描述：插件用来在gulp任务中开启node服务。nodemon常用的参数script: 启动的文件。
ext: 需要监听的文件，如果修改了则重启服务；ignore: 需要忽略的文件; 
重启需要开启的gulp任务。另外nodemon返回的是一个node流，支持on绑定事件。事件目前支持start
restart、crash等事件。
gulp.task('develop', function() {
    var stream = nodemon({
        script: 'app.js',
        ext: 'html js',
        ignore: ['ignored.js'],
        tasks: ['lint']
    })
    stream.on('start', function() {
        console.log('restarted')
    })
    .on('crash', function() {
        console.log(1234)
        stream.emit('start', 3);
    })
})

// run-sequence
run-sequence插件是用来按一定顺序运行gulp任务，其中参数你可以通过数组的形式来并行运行一些task任务
gulp.task('dist', function (cb) {
    runSequence(
        'clean',
        [ 'nunPrecompile', 'less', 'sprite', 'prod:weex' ],
        [ 'copy:before', 'weex:openSdk' ],
        [ 'transport', 'imagemin' ],
        [ 'concat', 'rev:img' ],
        'alias',
        [ 'usemin', 'copy:others', 'copy:base' ],
        [ 'move:css' ],
        [ 'replace', 'clean:css' ],
        cb
    );
});

// gulp-open
gulp.task('open', function() {
    gulp.src('')
    .pipe(open({
        uri: 'http://www.baidu.com'
    }))
})
```

