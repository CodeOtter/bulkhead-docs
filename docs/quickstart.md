# Quick Start

This guide will help you quickly integrate Bulkhead functionality into your project.

## Installation

In your SailsJS project directory, type the following:

`
npm install bulkhead
`

In the `config/bootstrap.js` file of your SailsJS project directory, replace the default `cb()` with:

```javascript
require('bulkhead').plugins.initialize(sails, cb);
```

## Services

Isolating your applications logic into services with Bulkhead will give you the following benefits:

* Attach common CRUD methods to a service to quickly access a model.
* Provides bulk updating, bulk deleting, and datatype-driven criteria searching of models.
* Standardizes service method responses so your service can be easily used for job distribution via message queues.
* Provides the option to convert a service into a promise chain via [Bluebird](https://github.com/petkaantonov/bluebird).

But first, we have to [install Bulkhead](quickstart.md#installation).

Next, in the `api/services` folder of your SailsJS project directory, create a new service called `AccountService.js` and populate it with:

```javascript
// api/services/AccountService.js

var Bulkhead = require('bulkhead');

module.exports = new function() {
    // In this example, we are assuming you have a model called api/models/Account.js
    // You can replace it with any model in your project.
    Bulkhead.service.call(this, 'Account');
};
```

Your service can now do the following:

```javascript
var AccountService = require('./api/services/AccountService.js');

// Finds a Person with an ID of 1
AccountService.find(1, done);

// Finds a Person with a first name of 'bob'
AccountService.find('bob', done);

// Finds a Person with a first name of 'bob' and a last name of 'smith'
AccountService.find({ firstName: 'bob', lastName: 'smith' }, done);

// Allows for criteria batching by finding a Person with a first name of 'bob', a last name of 'smith', and an ID of 1.
AccountService.find([ 1, 'bob', { lastName: 'smith' }], done);

// Creates a person with a first name of 'bob' and a last name of 'smith'
AccountService.create({ firstName: 'bob', lastName: 'smith' }, validation, done);

// Changes the first name to 'john' of the Person with an ID of 1
AccountService.update(1, { firstName: 'john' }, validation, done);

// Changes the first name to 'john' of all Persons with an ID of 1, the first name of 'bob', and the last name of 'smith' (Criteria batching)
AccountService.update([ 1, 'bob', { lastName: 'smith' }], { firstName: 'john' }, validation, done);

// Removes a Person with an ID of 1
AccountService.remove(1, done);

// Removes all Persons with an ID of 1, the first name of 'bob', and the last name of 'smith' (Criteria batching)
AccountService.remove([ 1, 'bob', { lastName: 'smith' }], done);

// Creates CRUD methods as promises.
AccountService.asPromise();

// Creates a person with a first name of 'bob' and a last name of 'smith'. (Criteria batching also possible)
AccountService.createPromise({ firstName: 'bob', lastName: 'smith' }).then(done);

// Changes the first name to 'john' of the Person with an ID of 1 (Criteria batching also possible)
AccountService.updatePromise(1, { firstName: 'john' }, validation).then(done);

// Removes a Person with an ID of 1 (Criteria batching also possible)
AccountService.removePromise(1).then(done);

// Finds a Person with an ID of 1 (Criteria batching also possible)
AccountService.findPromise(1).then(done);

// Generates a Bulkhead Response with a result of 1.  (See below)
AccountService.result([1]);

// Returns the Person model
AccountService.getModel();
```

> **NOTE:**
>
> **done** is a callback that is fired when the process has finished.  It passes the error in the first parameter, and the results in the second parameter.
>
> **validation** is a callback that is fired before the process begins.  It passes the instance of a model in the first parameter, and the callback to fire the process in the second parameter.

[Read this guide for more details about services](services.md).

## Testing

Quickly establishing a testing harness for your SailsJS project will help you deploy faster and smarter.  Bulkhead-Test uses the following:

* [Mocha](https://github.com/visionmedia/mocha) as the test harness
* [Barrels](https://github.com/bredikhin/barrels) for fixture loading
* [Supertest](https://github.com/visionmedia/supertest) for REST testing

First, we go to your SailsJS project directory and type:

```npm install bulkhead-test --saveDev```

Next, create a `test` folder and a `test/fixtures` folder in your SailsJS project directory.  Assuming we have a basic Account model in `api/models/Account.js`, let's create a new fixture  called `account.js` and populate it with:

```
[
  {
    "name": "Bob",
    "email": "bob@bob.com",
    "password": "Bob"
  },
  {
    "name": "Tim",
    "email": "tim@tim.com",
    "password": "Tim"
  },
  {
    "name": "Ed",
    "email": "ed@ed.com",
    "password": "Ed"
  }
]
```

> **NOTE:**
> 
> When test are ran, the account table will be cleared, and this data will be automatically inserted into the account table.

Next, let's prepare our test by creating `test/account.js` and populate it with the following:

```javascript
var suite = require('bulkhead-test'),
    assert = require('assert');

// This lifts the SailsJS application and prepares the testing harness
suite.lift();
  
describe('The Testing Harness', function() {
  describe('Accounts', function() {
    it('should find the fixture', function(done) {
      Account.findOneByName('Ed', function(err, result) {
        if(err) assert.fail();
        assert.equals(result.email, 'ed@ed.com');
        done();
      });
    })
  });
})
```

[Read this guide for more details about testing](testing.md).

## Create Plugin

First, we have to [install Bulkhead](quickstart.md#installation).

## Convert Plugin

First, we have to [install Bulkhead](quickstart.md#installation).
