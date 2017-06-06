# loglevel-plugin-remote
Plugin for sending [loglevel](https://github.com/pimterry/loglevel) messages to a remote log server

# Features
- Sends asynchronously and does not slow down the application
- Messages are sent one by one, so the order is maintained
- Support any server that accepts a Post request
- Support string substitutions like console and node.js (%s, %d, %j, %o)

## Installation

```sh
npm install loglevel-plugin-remote --save
```

## API

```javascript
apply(log[, options]);
```

**log** - root logger, imported from loglevel package

**options** - configuration object

```javascript
var defaults = {
  url: window.location.origin + '/logger',
  call: true,
  timeout: 0,
  trace: ['trace', 'warn', 'error'],
  clear: 1
}
```

- **url** - URL of log server API
- **call** - if set to true, then the original loglevel method will be called
- **timeout** - timeout in milliseconds ([MDN](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest/timeout))
- **trace** - lots of levels for which to add the stack trace
- **clear** - the number of rows to clean stack trace

## Base usage

### Browser directly

Download [production version](https://raw.githubusercontent.com/kutuluk/loglevel-plugin-remote/master/dist/loglevel-plugin-remote.min.js)
and copy to your project folder
```html
<script src="loglevel.min.js"></script>
<script src="loglevel-plugin-remote.min.js"></script>

<script>
  remote.noConflict().apply(log);
  log.warn('message');
</script>
```

### ES6
```javascript

import log from 'loglevel';
import remote from 'loglevel-plugin-remote';

remote.apply(log);
log.warn('message');
```

### CommonJS
```javascript
var log = require('loglevel');
var remote = require('loglevel-plugin-remote');

remote.apply(log);
log.warn('message');
```

### AMD
```javascript
define(['loglevel', 'loglevel-plugin-remote'], function(log, remote) {
  remote.apply(log);
  log.warn('message');
});
```

## Example

Code
```javascript
var log = require('loglevel');
var remote = require('loglevel-plugin-remote');

log.setLevel('trace');

remote.apply(log);

log.info('Log levels:');
log.trace('trace');
log.debug('debug');
log.info('info');
log.warn('warn');
log.error('error');
```

Output in a log server
```
Log levels:
trace message
    at http://localhost/js/test.js:35:5
debug message
info message
warn message
    at http://localhost/js/test.js:38:5
error message
    at http://localhost/js/test.js:39:5
```

Code
```javascript
log.info('String substitutions: %% %t %s', 'one', 'two');
log.info('Number substitutions: %d %d %d %d', 16, 1e6, '16', '1e6');
```

Output in a log server
```
String substitutions: % %t one two
Number substitutions: 16 1000000 16 1000000
```

Code
```javascript
log.info('Object substitutions:');

function Rectangle(width, height) {
  this.width = width;
  this.height = height;
}
var object = new Rectangle(10, 10);
log.info('%s, %d, %o, %j', object, object, object, object, object);

var date = new Date();
log.info('date: %o', date);

var error = new Error('My error');
log.info('error: %o', error);

var string = 'My string';
log.info('string: %o', string);

var number = 123;
log.info('number: %o', number);

var bool = true;
log.info('boolean: %o', bool);

var array = [1, 2, 3];
log.info('array: %o', array);
```

Output in a log server
```
Object substitutions:
[object Object], NaN, Rectangle{"height":10,"width":10}, {"height":10,"width":10} [object Object]
date: Date<"2017-06-04T13:16:01.455Z">
error: Error{}
string: String<"My string">
number: Number<123>
boolean: Boolean<true>
array: Array[1,2,3]
```

## Multiple plugins

Code
```javascript
var log = require('loglevel');
var remote = require('loglevel-plugin-remote');
var prefix = require('loglevel-plugin-prefix');

log.setLevel('trace');

remote.apply(log);
prefix.apply(log);

var array = [1, 2, 3];
log.info('array: %o', array);
```

Output in a log server
```
[12:53:46] INFO: array: Array[1,2,3]
```
