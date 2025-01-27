esmock
======
[![npm version](https://badge.fury.io/js/esmock.svg)](https://badge.fury.io/js/esmock) [![Build Status](https://github.com/iambumblehead/esmock/workflows/nodejs-ci/badge.svg)][2] [![install size](https://packagephobia.now.sh/badge?p=esmock)](https://packagephobia.now.sh/result?p=esmock) [![downloads](https://badgen.now.sh/npm/dm/esmock)](https://npmjs.org/package/esmock)

**esmock provides native ESM import mocking for unit tests.** Use examples below as a quick-start guide or use the [descriptive and friendly esmock guide here.][10]

[10]: https://github.com/iambumblehead/esmock/wiki
[0]: http://www.bumblehead.com "bumblehead"
[1]: https://github.com/iambumblehead/esmock/workflows/nodejs-ci/badge.svg "nodejs-ci pipeline"
[2]: https://github.com/iambumblehead/esmock "esmock"



`esmock` must be used with node's experimental --loader
``` json
{
  "name": "give-esmock-a-star",
  "type": "module",
  "scripts": {
    "test-uvu": "node --loader=esmock ./node_modules/uvu/bin.js ./spec/",
    "test-ava": "ava --node-arguments=\"--loader=esmock\"",
    "test-mocha": "mocha --loader=esmock --no-warnings"
  }
}
```

`esmock` has the following signature
``` javascript
await esmock(
  './to/module.js', // path to target module being tested
  { ...childmocks }, // mock definitions imported by target module
  { ...globalmocks } // mock definitions imported everywhere
);
```

`esmock` demonstrated with `ava` unit test examples
``` javascript
import test from 'ava';
import esmock from 'esmock';

test('should mock local files and packages', async t => {
  const main = await esmock('../src/main.js', {
    stringifierpackage : JSON.stringify,
    '../src/hello.js' : {
      default : () => 'world',
      exportedFunction : () => 'foobar'
    }
  });

  t.is(main(), JSON.stringify({ test : 'world foobar' }));
});

test('should do global instance mocks —third param', async t => {
  const { getFile } = await esmock('../src/main.js', {}, {
    fs : {
      readFileSync : () => 'returns this globally';
    }
  });

  t.is(getFile(), 'returns this globally');
});

test('should mock "await import()" using esmock.p', async t => {
  // using esmock.p, mock definitions are kept in cache
  const doAwaitImport = await esmock.p('../awaitImportLint.js', {
    eslint : { ESLint : cfg => cfg }
  });

  // mock definition is there, in cache, when import is called
  t.is(await doAwaitImport('cfg'), 'cfg');

  esmock.purge(doAwaitImport); // clear cache, if you wish
});

test('should merge "default" value, when safe', async t => {
  const main = await esmock('../src/main.js');

  t.is(main(), main.default()); // use the form you prefer
});

test('should use implicit "default"', async t => {
  const mainA = await esmock('../src/exportsMain.js', {
    '../src/main.js' : () => 'mocked main' // short-hand, smaller
  });
  const mainB = await esmock('../src/exportsMain.js', {
    '../src/main.js' : { default : () => 'mocked main' }
  });

  t.is(mainA(), mainB());
});
```
