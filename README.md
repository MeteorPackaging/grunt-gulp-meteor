grunt-meteor
============

When [submitting PRs to library authors to add Meteor integration](https://github.com/raix/Meteor-community-discussions/issues/14),
it helps to match their build process.

Most packages use the [Grunt](gruntjs.com) task runner and a `package.json`. To make it extra easy on the maintainers, below
are quasi-diffs that do Meteor testing and publishing.

They work by calling a bash script that does `meteor test-packages` and `meteor publish`. There is no direct Grunt
plugin to publish to Atmosphere, because this should be done through the `meteor` tool. Thus we'll use the
[grunt-shell](https://github.com/sindresorhus/grunt-shell) plugin to execute [those commands](https://github.com/RubaXa/Sortable/tree/master/meteor).

## Gruntfile.js

```js
    ...
    shell: {
      'meteor-test': {
        command: 'meteor/runtests.sh'
      },
      'meteor-publish': {
        command: 'meteor/publish.sh'
      }
    }

  ...
  grunt.loadNpmTasks('grunt-shell');

  ...
  // Meteor tasks
  grunt.registerTask('meteor-test', 'shell:meteor-test');
  grunt.registerTask('meteor-publish', 'shell:meteor-publish');
  // Ideally we'd run tests before publishing, but the chances of tests breaking (given that
  // Meteor is orthogonal to the library) are so small that it's not worth the maintainer's time
  // grunt.regsterTask('meteor', ['shell:meteor-test', 'shell:meteor-publish']);
  grunt.registerTask('meteor', 'shell:meteor-publish');
```

## Gruntfile.coffee

```coffee
    shell:
      'meteor-test':
        command: 'meteor/runtests.sh'
      'meteor-publish':
        command: 'meteor/publish.sh'

  grunt.loadNpmTasks 'grunt-shell'

  # Meteor tasks
  grunt.registerTask 'meteor-test', ['shell:meteor-test']
  grunt.registerTask 'meteor-publish', ['shell:meteor-publish']
  # Ideally we'd run tests before publishing, but the chances of tests breaking (given that
  # Meteor is orthogonal to the library) are so small that it's not worth the maintainer's time
  # grunt.regsterTask 'meteor', ['shell:meteor-test', 'shell:meteor-publish']
  grunt.registerTask 'meteor', 'shell:meteor-publish'
```

## package.json

The `grunt-shell` plugin should be part of `devDependencies` in `package.json`. Add the line below to that section:

    "grunt-shell": "*"
