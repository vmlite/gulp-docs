<!-- front-matter
id: watching-files
title: 文件监控
hide_title: true
sidebar_label: 文件监控
-->

# 文件监控

gulp api 中的 `watch()` 方法利用文件系统的监控程序（file system watcher）将 [globs][globs-docs] 与 [任务（task）][creating-tasks-docs] 进行关联。它对匹配 glob 的文件进行监控，如果有文件被修改了就执行关联的任务（task）。如果被执行的任务（task）没有触发 [异步完成][async-completion-doc] 信号，它将永远不会再次运行了。

此 API 的默认设置是基于通常的使用场景的，而且提供了内置的延迟和排队机制。

```js
const { watch, series } = require('gulp');

function clean(cb) {
  // body omitted
  cb();
}

function javascript(cb) {
  // body omitted
  cb();
}

function css(cb) {
  // body omitted
  cb();
}

exports.default = function() {
  // You can use a single task
  watch('src/*.css', css);
  // Or a composed task
  watch('src/*.js', series(clean, javascript));
};
```


## 警告：避免同步任务

就像注册到任务系统中的任务（task）一样，与文件监控程序关联的任务（task）不能是同步任务（synchronous taks）。如果你将同步任务（task）关联到监控程序，则无法确定任务（task）的完成情况，任务（task）将不会再次运行（假定当前正在运行）。

由于文件监控程序会让你的 Node 进程保持持续运行，因此不会有错误或警告产生。由于进程没有退出，因此无法确定任务（task）是否已经完成还是运行了很久很久了。

## 可监控的事件

默认情况下，只要创建、更改或删除文件，文件监控程序就会执行关联的任务（task）。 如果你需要使用不同的事件，你可以在调用 watch() 方法时通过 `events` 参数进行指定。可用的事件有 `'add'`, `'addDir'`, `'change'`, `'unlink'`, `'unlinkDir'`, `'ready'`, `'error'`。此外，还有一个 `'all'` 事件，它表示除 `'ready'` 和 `'error'` 之外的所有事件。

```js
const { watch } = require('gulp');

exports.default = function() {
  // All events will be watched
  watch('src/*.js', { events: 'all' }, function(cb) {
    // body omitted
    cb();
  });
};
```

## 初次执行

调用 watch() 之后，关联的任务（task）是不会被立即执行的，而是要等到第一次文件修之后才执行。

如需在第一次文件修改之前执行，也就是调用 watch() 之后立即执行，请将 `ignoreInitial` 参数设置为 `false`。

```js
const { watch } = require('gulp');

exports.default = function() {
  // The task will be executed upon startup
  watch('src/*.js', { ignoreInitial: false }, function(cb) {
    // body omitted
    cb();
  });
};
```

## 队列

`watch()` 方法能够保证当前执行的任务（task）不会再次并发执行。当文件监控程序关联的任务（task）正在运行时又有文件被修改了，那么所关联任务的这次新的执行将被放到执行队列中等待，直到上一次关联任务执行完之后才能运行。每一次文件修改只产生一次关联任务的执行并放入队列中。

如需禁止队列，请将 `queue` 参数设置为 `false`。

```js
const { watch } = require('gulp');

exports.default = function() {
  // The task will be run (concurrently) for every change made
  watch('src/*.js', { queue: false }, function(cb) {
    // body omitted
    cb();
  });
};
```


## 延迟

文件更改之后，只有经过 200 毫秒的延迟之后，文件监控程序所关联的任务（task）才会被执行。这是为了避免在同使更改许多文件时（例如查找和替换操作）过早启动任务（taks）的执行。

如需调整延迟时间，请为 `delay` 参数设置一个正整数。

```js
const { watch } = require('gulp');

exports.default = function() {
  // 文件第一次修改之后要等待 500 毫秒才执行关联的任务
  watch('src/*.js', { delay: 500 }, function(cb) {
    // body omitted
    cb();
  });
};
```

## 使用监控程序实例


你可能不会使用到此功能，但是如果你需要对被修改的文件进行完全的掌控 （例如访问文件路径或元数据）请使用从 `watch()` 返回的[chokidar][chokidar-module-package]。

__注意__： 返回的 chokidar 实例没有队列、延迟和异步完成（async completion）这些功能。

## Optional dependency

Gulp 有一个名为 [fsevents][fsevents-package] 的可选依赖项，他是一个特定于 Mac 系统的文件监控程序。如果你看到安装 fsevents 时出现的警告信息 (_"npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents"_) 这并不是什么问题，忽略即可。 如果跳过 fsevents 的安装，将使用一个备用文件监控程序，后续在 gulpfile 中产生的任何错误都将与此警告无关。

[globs-docs]: ../getting-started/6-explaining-globs.md
[creating-tasks-docs]: ../getting-started/3-creating-tasks.md
[async-completion-doc]: ../getting-started/4-async-completion.md
[chokidar-module-package]: https://www.npmjs.com/package/chokidar
[fsevents-package]: https://www.npmjs.com/package/fsevents
