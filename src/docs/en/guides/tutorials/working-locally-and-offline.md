---
title: Working locally & offline
description: 160 (or fewer) character description of this document!
sections:
  - Overview
  - Previewing vs. testing
  - Sandbox init scripts
  - HTTP testing
  - DB testing
---

## Overview

ADD ME!


## Previewing vs. testing

Arc targets two use cases:

1. **Previewing** - code runs locally and can be opened in a web browser
2. **Testing** - code runs headlessly in a terminal

Following the [quickstart](/en/guides/get-started/quickstart) you should have everything wired up so `npx sandbox` kicks up a local web server and creates tables and indexes defined in `.arc` for previewing work. 

If you want to write tests (and we very much think you should!) against the infra without deployment you'll need to set up the `sandbox` as a module.

This guide will use the following example `.arc` file:

```bash
@app
testapp

@http
get /api/cats
get /api/cats/:catID
post /api/cats
put /api/cats/:catID
delete /api/cats/:catID

@tables
cats
  pplID *String
  catID **String

ppl
  pplID *String

@indexes
ppl
  email *String
```
---

## Sandbox init scripts

Set up a test suite:

```bash
mkdir tests
npm i tape tap-spec --save-dev
touch tests/http-test.js
touch tests/db-test.js
```

Add the following to `package.json`:

```javascript
{
  "test": "NODE_ENV=testing AWS_PROFILE=xxx AWS_REGION=xxx tape tests/*-test.js | tap-spec"
}
```

**🖖 Important!**

- Setup `AWS_PROFILE` and `AWS_REGION` per the [installation guide](/en/guides/aws/configuration)
- `NODE_ENV=testing` is VERY important to add to your `npm test` scripts regardless of the test framework you are using--- you MUST include it or you will have a bad time

> ✨ Tip: while you can use any test runner and reporter combo you want, the TAP family is strongly recommended; test suites that require test runners to inject globals create difficult-to-debug situations

Scaffold your two test files with an environment check; this a good practice to get the testing muscles warmed up.

```javascript
// tests/http-test.js
var test = require('tape')
var arc = require('@architect/architect')

test('env', t=> {
  t.plan(1)
  t.ok(arc.sandbox.http, 'arc.sandbox.http exists in current scope')
})
```

```javascript
// tests/db-test.js
var test = require('tape')
var arc = require('@architect/architect')

test('env', t=> {
  t.plan(1)
  t.ok(arc.sandbox.db, 'arc.sandbox.db exists in current scope')
})
```

Check the tests by running `npm t`. (It's ok if things fail &mdash; that's exactly why we have tests!)

---

## HTTP testing

In order to test HTTP routes we will need an HTTP client. Lets use [tiny-json-http](https://github.com/brianleroux/tiny-json-http), a small, dependency free module with a straightforward interface. Install by running `npm i tiny-json-http --save-dev` and edit the HTTP test:

```javascript
// tests/http-test.js
var test = require('tape')
var tiny = require('tiny-json-http')
var arc = require('@architect/architect')

test('env', t=> {
  t.plan(1)
  t.ok(arc.sandbox.http, 'arc.sandbox.http exists in current scope')
})

/**
 * first we need to start the local http server
 */
let close
test('arc.sandbox.start', async t=> {
  t.plan(1)
  close = await arc.sandbox.start()
  t.ok(true, 'http server started on http://localhost:3333')
})

/**
 * then we can make a request to it and check the result
 */
test('get /', async t=> {
  t.plan(1)
  let url = 'http://localhost:3333'
  let result = await tiny.get({url})
  t.ok(result.body, 'got 200 response')
})

/** 
 * finally close the server so we cleanly exit the test
 */
test('server.close', t=> {
  t.plan(1)
  close()
  t.ok(true, 'server closed')
})
```

As your app matures you will want to augment these tests with more elaborate response checks.

---

## DB testing

In an `.arc` defined project `NODE_ENV` is used for knowing where the code is running. This way apps with `NODE_ENV` set to `staging` or `production` will load the correct DynamoDB endpoints. Your test suite and any client wrappers you author should follow suit.

```javascript
// tests/db-test.js
var AWS = require('aws-sdk')
var endpoint = new AWS.Endpoint('http://localhost:5000')
var db = process.env.NODE_ENV === 'testing'? new AWS.DynamoDB({endpoint}) : new AWS.DynamoDB

var test = require('tape')
var arc = require('@architect/architect')

/**
 * first we need to start the local db server and grab a reference to the client
 */
var client 
test('arc.sandbox.db.start', t=>{
  t.plan(1)
  client = arc.sandbox.db.start(xxx=> t.ok(true, 'started'))
})

/**
 * then we can work with the db using the vanilla `DynamoDB` client (or `DynamoDB.DocumentClient`)
 */
test('db', t=> {
  t.plan(1)
  // note: we do not need to create the tables the
  // sandbox detected the .arc and did that above
  db.listTables({}, function _list(err, result) {
    if (err) throw err
    t.ok(result, 'got result')
    console.log(result) 
  })
})

/** 
 * finally close the db client so we cleanly exit the test
 */
test('arc.sandbox.db.close', t=>{
  t.plan(1)
  // finally we'll use that client reference from above to close the sandbox
  client.close()
  t.ok(true, 'closed')
})
```
> 🔭 Get the source for this example [architect/arc-example-working-locally](https://github.com/architect/arc-example-working-locally)

