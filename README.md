grunt-meteor
============

When [submitting PRs to library authors to add Meteor integration](https://github.com/raix/Meteor-community-discussions/issues/14),
it helps to match their build process.

Most packages use the [Grunt](http://gruntjs.com) task runner and a `package.json`. To make it extra easy on the maintainers, below are quasi-diffs that do Meteor testing and publishing.

There is no direct Grunt plugin to publish to Atmosphere, because this should be done through the `meteor` tool. We'll execute `meteor publish` using the [grunt-exec](https://github.com/jharding/grunt-exec) plugin.

For testing, we'll use [spacejam](http://stackoverflow.com/questions/27209779/exit-meteor-tinytest-after-after-all-tests-have-been-completed), which in turn executes `meteor test-packages`.

Add the lines below in the corresponding sections in the Gruntfile. For your convenience, there are 2- and 4-space indented sections, and a .coffee file.

## package.json

First, you need to add the two `devDepencencies` in `package.json`:

    "grunt-exec": "^0.4.6"
    "spacejam": "^1.1.1"

It may be tempted to [add Meteor as a sort of devDependency](https://github.com/MeteorPackaging/grunt-meteor/issues/1#issuecomment-65001441), but most of the time, the original author won't deal with Meteor; as for Meteor devs, they already have Meteor. Thus we'll install Meteor if it's not installed already, from the Grunt file. It has also been suggested to [just check if Meteor is installed and print a warning otherwise](https://github.com/MeteorPackaging/grunt-meteor/issues/1#issuecomment-65030301) when running `meteor` tasks, instead of automagically installing it.

## Gruntfile.js - 2-space indentation

```js

    // if there's a "clean" task, add Meteor to it; otherwise, see meteor-cleanup below
    clean: {
      ...
      meteor: ['.build.*', 'versions.json', 'package.js']
    }
    
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
      // !- only add this if there was no "clean" task
      'meteor-cleanup': {
        // remove build files and package.js
        command: 'rm -rf .build.* versions.json package.js'
      },
      'meteor-test': {
        command: 'node_modules/.bin/spacejam --mongo-url mongodb:// test-packages ./'
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
        // if there's a "clean" task, add Meteor to it; otherwise, see meteor-cleanup below
        clean: {
            ...
            meteor: ['.build.*', 'versions.json', 'package.js']
        }
    
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
            // !- only add this if there was no "clean" task
            'meteor-cleanup': {
                // remove build files and package.js
                command: 'rm -rf .build.* versions.json package.js'
            },
            'meteor-test': {
                command: 'node_modules/.bin/spacejam --mongo-url mongodb:// test-packages ./'
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

    # if there's a "clean" task, add Meteor to it; otherwise, see meteor-cleanup below
    clean:
      ...
      meteor: ['.build.*', 'versions.json', 'package.js']

    ...
    
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
      # !- only add this if there was no "clean" task
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

## .travis.yml

We can include Meteor tests in the continuous integration file, [.travis.yml](https://github.com/MeteorPackaging/hammer.js/blob/master/.travis.yml):

```yml
language: node_js
node_js:
  - "0.10"

before_script:
  - npm install -g grunt-cli
  # Install Meteor preemptively because installing via grunt-exect has failed in the past
  - curl https://install.meteor.com | /bin/sh
  
script:
  - grunt test-travis
  - grunt meteor-test
```

Regarding the preemptive installation of Meteor, check [this Travil build log for hammer.js](https://travis-ci.org/MeteorPackaging/hammer.js/builds/42705466). Installing Meteor via `grunt-exec` has failed for no clear reason, but installing via `.travis.yml` hasn't yet (ever), so be safe. On the other hand, it's possible that that particular grunt-exec failure was caused by a transient network error.
