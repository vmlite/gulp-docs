<!-- front-matter
id: working-with-files
title: 文件处理
hide_title: true
sidebar_label: 文件处理
-->

# 文件处理

gulp 暴露了 `src()` 和 `dest()` 方法用于处理计算机上存放的文件。

`src()` 接受 [glob][explaining-globs-docs] 参数，并从文件系统中读取文件然后生成一个 [Node stream][node-streams-docs]。它将所有匹配的文件读取到内存中并通过流（stream）进行处理。

由 `src()` 产生的流（stream）应当从任务（task）中返回并指出异步完成，就如 [创建任务（task）[creating-tasks-docs] 一中所述。

```js
const { src, dest } = require('gulp');

exports.default = function() {
  return src('src/*.js')
    .pipe(dest('output/'));
}
```

流（stream）所提供的主要的 API 是 `.pipe() `方法，用于连接转换流（Transform streams）或可写流（Writable streams）。

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(dest('output/'));
}
```


`dest()` 接受一个输出目录作为参数，并且它还会产生一个[Node stream][node-streams-docs]，通常作为终止流（terminator stream）。当它接收到通过管道（pipeline）传输的文件时，它会将文件内容及文件属性写入到指定的目录中。gulp 还提供了 [`symlink()`][symlink-api-docs] 方法，其操作方式类似 `dest()`，但是创建的是链接而不是文件。

大多数的插件都除以`src()`和`dest()`之间，使用`.pipe()`方法处理流(Stream)中的文件。

## 向流（stream）中添加文件

`src()` 也可以放在管道（pipeline）的中间，以根据给定的 `globs` 向流（stream）中添加文件。新加入的文件只对后续的转换可用。如果 [glob 匹配的文件与之前的有重复][overlapping-globs-docs]，仍然会再次添加文件。

这对于在添加普通的 JavaScript 文件之前先转换部分文件的场景很有用，添加新的文件后可以对所有文件统一进行压缩并混淆（uglifying）。

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(src('vendor/*.js'))
    .pipe(uglify())
    .pipe(dest('output/'));
}
```

## 分阶段输出

`dest()` 可以用在管道（pipeline）中间用于将文件的中间状态写入文件系统。当接收到一个文件时，当前状态的文件将被写入文件系统，文件路径也将被修改以反映输出文件的新位置，然后该文件继续沿着管道（pipeline）传输。

此功能可用于在同一个管道（pipeline）中创建未压缩（unminified）和已压缩（minified）的文件。

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(src('vendor/*.js'))
    .pipe(dest('output/'))
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}
```

## 模式：流（streaming）、缓冲（buffered）和空（empty）

`src()` 可以工作在三种模式下：缓冲（buffering）、流（streaming）和空（empty）模式。这些模式可以通过对 `src()` 的 `buffer` 和 `read` [参数][src-options-api-docs] 进行设置。

* 缓冲（Buffering）模式是默认模式，将文件内容加载内存中。插件通常运行在缓冲（buffering）模式下，并且许多插件不支持流动（streaming）模式。

* 流动（Streaming）模式的存在主要用于操作无法放入内存中的大文件，例如巨幅图像或电影。文件内容从文件系统中以小块的方式流式传输，而不是一次性全部加载。如果需要流动（streaming）模式，请查找支持此模式的插件或自己编写。

* 空（Empty）模式不包含任何内容，仅在处理文件元数据时有用。

[explaining-globs-docs]: ../getting-started/6-explaining-globs.md
[creating-tasks-docs]: ../getting-started/3-creating-tasks.md
[overlapping-globs-docs]: ../getting-started/6-explaining-globs.md#overlapping-globs
[node-streams-docs]: https://nodejs.org/api/stream.html
[symlink-api-docs]: ../api/symlink.md
[src-options-api-docs]: ../api/src.md#options
