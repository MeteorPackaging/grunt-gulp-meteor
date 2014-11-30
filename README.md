grunt-meteor
============

When [submitting PRs to library authors to add Meteor integration](https://github.com/raix/Meteor-community-discussions/issues/14),
it helps to match their build process.

Most packages use the [Grunt](gruntjs.com) task runner and a `package.json`. To make it extra easy on the maintainers, below
are quasi-diffs that do Meteor testing and publishing.

There is no direct Grunt plugin to publish to Atmosphere, because this should be done through the `meteor` tool. We'll execute `meteor publish` using the [grunt-exec](https://github.com/jharding/grunt-exec) plugin.

For testing, we'll use [spacejam](stackoverflow.com/questions/27209779/exit-meteor-tinytest-after-after-all-tests-have-been-completed), which in turn executes `meteor test-packages`.

Add the lines below in the corresponding sections in the Gruntfile. For your convenience, there are 2- and 4-space indented sections, and a .coffee file.

## package.json

First, you need to add the two `devDepencencies` in `package.json`:

    "grunt-exec": "^0.4.6"
    "spacejam": "^1.1.1"


## Gruntfile.js - 2-space indentation

```js

    ...
    exec: {
      'meteor-init': {
        command: [
          // Make sure Meteor is installed, per https://meteor.com/install.
          // The curl'ed script is safe; takes 2 minutes to read source & check.
          'type meteor >/dev/null 2>&1 || { curl https://install.meteor.com/ | sh; }',
          // Meteor expects package.js to be in the root directory of
          // the checkout, so copy it there temporarily
          'cp meteor/package.js .'
        ].join(';')
      },
      'meteor-cleanup': {
        // remove build files and package.js
        command: 'rm -rf .build.* versions.json package.js'
      },
      'meteor-test': {
        command: 'spacejam --mongo-url mongodb:// test-packages ./'
      },
      'meteor-publish': {
        command: 'meteor publish'
      }
    }


  ...
  // !- add the line below ONLY if you see other grunt.loadNpmTasks() calls
  grunt.loadNpmTasks('grunt-exec');
  // !- you DON'T need to add the line above if you see: require('load-grunt-tasks')(grunt);

  ...
  // Meteor tasks
  grunt.registerTask('meteor-test', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-cleanup']);
  grunt.registerTask('meteor-publish', ['exec:meteor-init', 'exec:meteor-publish', 'exec:meteor-cleanup']);
  grunt.registerTask('meteor', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-publish', 'exec:meteor-cleanup']);
```

## Gruntfile.js - 4-space indentation

```js
        exec: {
            'meteor-init': {
                command: [
                    // Make sure Meteor is installed, per https://meteor.com/install.
                    // The curl'ed script is safe; takes 2 minutes to read source & check.
                    'type meteor >/dev/null 2>&1 || { curl https://install.meteor.com/ | sh; }',
                    // Meteor expects package.js to be in the root directory of
                    // the checkout, so copy it there temporarily
                    'cp meteor/package.js .'
                ].join(';')
            },
            'meteor-cleanup': {
                // remove build files and package.js
                command: 'rm -rf .build.* versions.json package.js'
            },
            'meteor-test': {
                command: 'spacejam --mongo-url mongodb:// test-packages ./'
            },
            'meteor-publish': {
                command: 'meteor publish'
            }
        }


    ...
    // !- add the line below ONLY if you see other grunt.loadNpmTasks() calls
    grunt.loadNpmTasks('grunt-exec');
    // !- you DON'T need to add the line above if you see: require('load-grunt-tasks')(grunt);


    ...
    // Meteor tasks
    grunt.registerTask('meteor-test', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-cleanup']);
    grunt.registerTask('meteor-publish', ['exec:meteor-init', 'exec:meteor-publish', 'exec:meteor-cleanup']);
    grunt.registerTask('meteor', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-publish', 'exec:meteor-cleanup']);

```

## Gruntfile.coffee

```coffee
    exec:
      'meteor-init':
        command: [
          # Make sure Meteor is installed, per https://meteor.com/install.
          # The curl'ed script is safe; takes 2 minutes to read source & check.
          'type meteor >/dev/null 2>&1 || { curl https://install.meteor.com/ | sh; }',
          # Meteor expects package.js to be in the root directory
          # of the checkout, so copy it there temporarily
          'cp meteor/package.js .'
        ].join(';')
      'meteor-cleanup':
        # remove build files and package.js
        command: 'rm -rf .build.* versions.json package.js'
      'meteor-test':
        command: 'spacejam --mongo-url mongodb:// test-packages ./'
      'meteor-publish':
        command: 'meteor publish'

  ...
  # !- add the line below ONLY if you see other grunt.loadNpmTasks() calls
  grunt.loadNpmTasks 'grunt-exec'
  # !- you DON'T need to add the line above if you see: require('load-grunt-tasks')(grunt);

  ...
  # Meteor tasks
  grunt.registerTask 'meteor-test', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-cleanup']
  grunt.registerTask 'meteor-publish', ['exec:meteor-init', 'exec:meteor-publish', 'exec:meteor-cleanup']
  grunt.registerTask 'meteor', ['exec:meteor-init', 'exec:meteor-test', 'exec:meteor-publish', 'exec:meteor-cleanup']
```

