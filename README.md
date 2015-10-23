[![Build Status](https://travis-ci.org/mozilla/addons-validator.svg?branch=master)](https://travis-ci.org/mozilla/addons-validator)
[![Coverage Status](https://coveralls.io/repos/mozilla/addons-validator/badge.svg?branch=master&service=github)](https://coveralls.io/github/mozilla/addons-validator?branch=master)
[![Dependency Status](https://david-dm.org/mozilla/addons-validator.svg)](https://david-dm.org/mozilla/addons-validator)
[![devDependency Status](https://david-dm.org/mozilla/addons-validator/dev-status.svg)](https://david-dm.org/mozilla/addons-validator#info=devDependencies)

# addons-validator

The Add-ons Validator, JS edition.

## Usage

### Command Line

You need node.js to use the add-ons validator.

To validate your add-on locally, install the validator from
[npm](http://nodejs.org/):

```
# Install locally so you can use the validator from anywhere on your machine.
npm install -g addons-validator
```

After installation, run the validator and direct it to your add-on file:

```
addons-validator my-addon.xpi
```

The addons-validator will check your add-on and show you errors, warnings,
and friendly messages for your add-on. If you want more info on the options
you can enable/disable for the command-line app, use the `--help` option:

```
addons-validator --help
```

## Development

If you'd like to help us develop the addons-validator, that's great! It's
pretty easy to get started, you just need node.js installed on your machine.

### Required node version

addons-validator requires node.js v0.12.x or greater. Using nvm is probably the
easiest way to manage multiple node versions side by side. See
[nvm on github](https://github.com/creationix/nvm) for more details.

### Install dependencies

Install dependencies with [npm](http://nodejs.org/):

```
npm install
```

Dependencies are automatically kept up-to-date using [greenkeeper](http://greenkeeper.io/).

### npm scripts and grunt tasks

Run basic automation tasks via npm scripts (e.g. `npm test`).
These don't need `grunt-cli` installed globally.

#### npm scripts

| Script       | Description                                               |
|--------------|-----------------------------------------------------------|
| npm test     |  Runs the tests                                           |
| npm start    |  Builds the lib and watches for changes                   |

If you install `grunt-cli` globally then you can run other tasks.

```
npm install -g grunt-cli
```

From the grunt docs:

>  The job of the Grunt CLI is simple: run the version of Grunt which has
   been installed next to a Gruntfile. This allows multiple versions of
   Grunt to be installed on the same machine simultaneously.

#### Grunt tasks

| Script                 | Description                                      |
|------------------------|--------------------------------------------------|
| grunt test             |  Runs the tests                                  |
| grunt test-no-coverage |  Runs the tests (without coverage)               |
| grunt build            |  Builds the lib                                  |
| grunt start            |  Builds the lib and watches for changes          |
| grunt eslint           |  Lints the files with eslint (Run in grunt test) |
| grunt jscs             |  Checks for style issues  (Run in grunt test)    |


### Building and watching for changes

You can run `npm start` to build the library and then rebuild on file changes.

Once you build the library you can use the CLI in `bin/addons-validator`.

### Testing

Tests use `grunt` but don't require global `grunt`. Just run `npm test`.

#### Coverage

We're looking to maintain coverage at 100%. Use the coverage data in the
test output to work out what lines aren't covered and ensure they're covered.

#### Testing and promises

Tests using promises should return the promise. This removes the need to call
`done()` in your tests:

```javascript
it('should do something promise-y', () => {
  return somePromiseCall()
    .then(() => {
      // Assert stuff here.
    });
})
```

To test for rejection you can use this pattern:

```javascript
import { unexpectedSuccess } from 'utils';
...
it('should reject because of x', () => {
  return somePromiseCall()
    .then(unexpectedSuccess)
    .catch((err) => {
      // make assertions about err here.
    });
})
```

#### Assertions and testing APIs

`assert`, `describe`, `it`, `beforeEach`, and `afterEach` are
available in tests by default–you don't need to import anything
for those to work.

We're using chai for assertions [see the Chai docs for the API
available](http://chaijs.com/api/assert/)

### Logging

We use [bunyan](https://github.com/trentm/node-bunyan) for logging:

* By default logging is off (level is set to 'fatal') .
* Logging in tests can be enabled using an env var e.g: `LOG_LEVEL=debug grunt test`
* Logging on the cli can be enabled with `--log-level [level]`.
* Bunyan by default logs JSON. If you want the json to be pretty printed
  pipe anything that logs into `bunyan` e.g. `LOG_LEVEL=debug grunt test
  | node_modules/bunyan/bin/bunyan`


## Architecture

In a nutshell the way the validator works is to take an add-on
package, extract the metadata from the xpi (zip) format and then
process the files it finds through various content scanners.

![Architecture diagram](https://raw.github.com/mozilla/addons-validator/master/docs/diagrams/addon-validator-flow.png)

### Scanners

Each file-type has a scanner. For example: CSS files use `CSSScanner`;
Javascript files use `JavaScriptScanner`. Each scanner looks at relevant
files and passes each file through a parser which then hands off to
a set of rules that look for specific things.

### Rules

Rules get exported via a single function in a single file. A rule can
have private functions it uses internally, but rule code should not depend
on another rule file and each rule file should export one rule.

Each rule function is passed data from the scanner in order to carry
out the specific checks for that rule it returns a list of objects which
are then made into message objects and are passed to the Collector.

### Collector

The Collector is an in-memory store for all validation message objects
"collected" as the contents of the package are processed.

### Messages

Each message has a code which is also its key. It has a message which
is a short outline of what the message represents, and a description
which is more detail into why that message was logged. The type of
the message is set as messages are added so that if necessary the
same message could be an error *or* a warning for example.

### Output

Lastly when the processing is complete the validator will output
the collected data as text or JSON.
