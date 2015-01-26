To integrate Meteor tasks into a Gulp file, add the following lines, or ensure they exist:

```js
var exec = require('child_process').exec;

function execCallback(error, stdout, stderr) {
  if (error !== null) console.error(error);
}

gulp.task('meteor-init', function () {
  // Meteor expects package.js to be in the root directory
  // of the checkout, so copy it there temporarily
  exec('cp meteor/package.js .', execCallback);
});

gulp.task('meteor-test', ['meteor-init'], function () {
  exec('node_modules/.bin/spacejam --mongo-url mongodb:// test-packages ./', execCallback);
  exec('rm -rf .build.* versions.json package.js', execCallback);  // cleanup
});

gulp.task('meteor-publish', ['meteor-init'], function () {
  exec('meteor publish && rm -rf .build.* versions.json package.js', execCallback);
});
```

See [the main README](/README.md) for more details.
