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

First, we have to [install Bulkhead](quickstart.md#installation).

## Create Plugin

First, we have to [install Bulkhead](quickstart.md#installation).

## Convert Plugin

First, we have to [install Bulkhead](quickstart.md#installation).
