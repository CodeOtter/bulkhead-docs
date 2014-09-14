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

```npm install bulkhead-test --save-dev```

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

describe('The Testing Harness', function() {
  // This lifts the SailsJS application and prepares the testing harness
  suite.lift();

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

## Dependency Injection
_(Experimental)_

Dependency Injection provides developers with a way to override internal functionality of a service without having to modify the code of that service.  For example, assuming you had a package called ``mailerPlugin`` and the main service performed mailing, it would look like this:

```javascript
var Mailer = require('mailerPlugin');
var mailer = new Mailer();
mailer.dispatch('to@mail.com', 'from@mail.com', 'Test body');
```

Let's assuming that, by default, ```mailerPlugin``` used ```nodeMailer``` to handle its mailing functionality.  Now let's assume you have a library for mailing functionality called ```customMailer``` (_and it has all of the same methods as ```nodeMailer```_) and you wanted ```mailerPlugin``` to use that instead, you would do the following:

```javascript
var Mailer = require('mailerPlugin');
var mailer = new Mailer({
    'mailer': require('customMailer')
});
mailer.dispatch('to@mail.com', 'from@mail.com', 'Test body');
```

This will override ```nodeMailer``` with ```customMailer``` in the ```mailerPlugin``` without having to branch or modify the code of the ```mailerPlugin``` itself.

To create a service with such flexibility, we do the following:

```javascript
// api/services/someService.js
var Bulkhead = require('bulkhead');

module.exports = Bulkhead.dic(function(container) {
    Bulkhead.service.call(this); // Build a service as you would normally

    console.log(container.get('someDep')) // This will output 7 when the service is instantiated
}, {
    'someDep': 7
});
```

To instantiate that service, you'd do the following:

```javascript
var SomeService = require('./api/services/someService');
var service = new SomeService();
```

This will output ```7``` to the console.

To override the ```someDep``` dependency, you'd instantiate the service like this:

```javascript
var SomeService = require('./api/services/someService');
var service = new SomeService({
    'someDep': 8
});
```

This will output ```8``` to the console.

Assuming we had a service with parameters in the constructor:

```javascript
// api/services/anotherService.js
module.exports = Bulkhead.dic(function(a, b, container) {
	Bulkhead.service.call(this); // Build a service as you would normally

	console.log(a);
	console.log(b);
	console.log(container.get('someDep')) // This will output 7 when the service is instantiated
}, {
    'someDep': 7
});
```

We would instantiate the service and override the dependency like this:

```javascript
var SomeService = require('./api/services/anotherService');
var service = new SomeService(6, 7, {
    'someDep': 8
});
```

This will output ``` 6 7 8``` to the console.

However, if we didnt' want to override the dependecy, we would instantiate the service like this:

```javascript
var SomeService = require('./api/services/anotherService');
var service = new SomeService(6, 7);
```

This will output ``` 6 7 7``` to the console.

[Read this guide for more details about dependency injection](dic.md).

## Create Project
_(Experimental)_

Bulkhead also provides a way to build a brand new SailsJS project with Bulkhead already integrated.

Simply run the following:

```
wget -qO- https://raw.githubusercontent.com/CodeOtter/bulkhead/develop/createProject | bash
```

Then follow the prompts.  This will do the following:

* Create a new Github repository for your SailsJS project.
* Create a brand new SailsJS 0.10.4 application
* Create a `master` and `develop` branch in the repository.
* Create a `LICENSE` file.
* Create a `README.md` stub with installation instructions.
* Create a `.gitignore` file.
* Create a stub test in the `test` directory.
* Populate the `package.json` for NPM packaging.
* Install the `bulkhead` and `bulkhead-test`, `connect-redis@1.4.5`, and `sails-mysql` packages.
* Setup environmental configurations for `production`, `development`, and `test`.
* Provide you with instructions on how to move forward when you are ready to release.
* Configure testing for the project to be ran via `npm test`

[Read this guide for more details about projects](project.md).


## Create Plugin

Isolating related processes to an NPM package is a common task in NodeJS.  At this time, SailsJS does not have a fairly comprehensive way to utilize NPM packages as plugins.  However, with Bulkhead, its possible to treat NPM packages as SailsJS plugins *without* having to modify SailsJS core code.

First, we have to [install Bulkhead](quickstart.md#installation).

Then, we change directory to your web/development root (the parent of your SailsJS project directory) and run the following:

```
wget -qO- https://raw.githubusercontent.com/CodeOtter/bulkhead/master/createPackage | bash
```

Then follow the prompts.  This will do the following:

* Create a new Github repository for your SailsJS plugin.
* Create a `master` and `develop` branch in the repository.
* Create a `LICENSE` file.
* Create a `README.md` stub with installation instructions.
* Create a `.gitignore` file.
* Create a stub test in the `test` directory.
* Create a stub service for you in the `api/services` directory.
* Populate the `package.json` for NPM package.
* Install the `bulkhead`, `bulkhead-test`, and `async` packages for your new package.
* Provide you with instructions on how to move forward when you are ready to release.
* Configure testing for the package to be ran via `npm test`
* Setup `index.js` to register itself in Bulkhead

[Read this guide for more details about plugins](plugin.md).

## Convert Plugin

Converting your established NPM package into a SailsJS plugin will give it a much wider audience of developers.

First, we have to [install Bulkhead](quickstart.md#installation).

To have an NPM package load as a SailsJS plugin, then do the following:

* Create an `api` folder in your package.  This folder should contain subfolders of `models`, `services`, `policies`, `adapters`, `controllers`, `hooks`, `blueprints`, and `responses` like a SailsJS project would.
* Create a `config` folder in your package.  This folder should contain the JavaScript files exporting JSON in the same fashion as a SailsJS project would.
* Create a `api/services/Service.js` file that exports an object that mixes in with a Bulkhead service via `Bulkhead.service.call(this)`; ([See services](quickstart.md#services))
* Create an `index.js` file, populated with ```require('bulkhead').plugins.register();```
* In the `package.json`, make sure the `main` property is set to `index.js`

That's it.  Once this package is installed via NPM, let's examine how to access it.

Assuming your NPM package is called `TestPackage` and has the following project directory structure:

```
testPackage
    |
    +- api
    |   |
    |   +- models
    |   |   |
    |   |   +- Account.js
    |   |
    |   +- services
    |       |
    |       +- TestService.js
    |       |
    |       +- OtherService.js
    |
    +- config
        |
        +- test.js
```

We can access this NPM package as a SailsJS plugin via a service or controller method in the following fashion:

```javascript
// SailsJSProject/api/services/AppService.js

var Bulkhead = require('bulkhead'),
    testPackage = require('testPackage');

module.exports = {

    someMethod : function() {

    	// These are all the ways to access the Account model
        testPackage.models.account;
        testPackage.Account;
        sails.models['testPackage_account'];
        global['testPackage_Account'];

		// These are all the ways to access the TestService
        testPackage.services.testservice;
        testPackage.TestService;
        sails.services['testPackage_testservice'];
        global['testPackage_TestService'];

        // These are all the ways to access the OtherService
        testPackage.services.otherservice;
        testPackage.OtherService;
        sails.services['testPackage_otherservice'];
        global['testPackage_OtherService'];
        
    }
};
```

In the database, all tables referred to in a plugin's models will be given namespaced prefixes, so to access the table in MySQL for example:

```
SELECT * FROM testPackage_account;
```

[Read this guide for more details about plugins](plugin.md).
