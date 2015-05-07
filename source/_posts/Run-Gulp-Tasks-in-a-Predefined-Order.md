title: "Run Gulp Tasks in a Predefined Order"
date: 2015-05-07 18:26:28
tags:
---

[Gulp] is build automation tool which runs on top of Node. In my current project I had to execute set of tasks in a predefined order, in which some of the tasks can be run in parallel to improve the efficiency.

By default [Gulp] runs all the tasks at the same time, so after googling I saw this official [article][1]. In this approach every task will takes a callback function so that Gulp knows when that task is completed. Then other task add this as a dependency

E.g. from the official article

```js
var gulp = require('gulp');

// takes in a callback so the engine knows when it'll be done
gulp.task('one', function(cb) {
    // do stuff -- async or otherwise
    // if err is not null and not undefined, the orchestration will stop, and 'two' will not run
    cb(err);
});

// identifies a dependent task must be complete before this one begins
gulp.task('two', ['one'], function() {
    // task 'one' is done now
});

gulp.task('default', ['one', 'two']);
// alternatively: gulp.task('default', ['two']);
```

Here task `two` is depends on task `one`, so Gulp ensures that task `one` is completed before running task `two`.One drawback with this approach is tasks are tightly coupled to each other which reduces the flexibility if you want to change the order.

So I came across a plugin called [run-sequence][2], this plugin allows you to order the tasks much easy. We can re-write the first example like this

```js
var gulp = require('gulp'),
    runSequence = require('run-sequence');

gulp.task('one', function() {
  // Returning the stream is improtant
  return gulp.src(....)
        .pipe(gulp.dest(....);
});

gulp.task('two',function() {
  // Returning the stream is improtant
  return gulp.src(....)
        .pipe(gulp.dest(....);
});

gulp.task('default', function(){
    runsSequence('one', 'two');
});

```

**Caution**
Make sure each of the tasks returns a `stream`

### Running tasks in parallel

[run-sequence][2] can do much more, like running in parallel

```js
gulp.task('default', function(){
    runsSequence(
        'task-1',
        'task-2',
        ['task-3', 'task-4', 'task-5'],
        ['task-7', 'task-8']
      );
});

```
The above one ensures that `task-1` and `task-2` will be running in series, after that `task-3`, `task-4`, `task-5` will run in parallel, then `task-7` and `task-8` are run in parallel as well.

Main advantage I see when compared to official documentation is - tasks are independent and orchestration is done by someone else.



[Gulp]: http://gulpjs.com/
[1]: https://github.com/gulpjs/gulp/blob/master/docs/recipes/running-tasks-in-series.md
[2]: https://www.npmjs.com/package/run-sequence
