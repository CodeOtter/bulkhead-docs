## Advanced Bulkhead Usage
--------------------------
### Datatype Criteria Mapping

Let's assume that when we search by a string, we want to search by last name and not first names.  Now lets say when we search by number, we wish to search by age instead of ID.

To override this default functionality, we define our criteria map at mixin time:

```javascript
var Bulkhead = require('bulkhead');

module.exports = new function() {
    // In this example, we are assuming you have a model called Person, but you can
    // replace it with any model in your project.
    var self = this;
	Bulkhead.service.call(this, 'Account', {
	    'number': function(criteria, next) {
	    	self.getModel.findByAge(criteria, next);
	    },
		'string': function(criteria, next) {
			self.getModel().findByLastName(criteria, next);
		}
	});
};
```

Now when we search by a number, it will search by age and when we search with a string, it will search by last name.

### Service Responses

To ease in preserving statefulness between stateless services, (For example, passing jobs, their immediate scope AND their arguments into a message queue) the methods of a Bulkhead service should always return a Response.  The ```.result()``` helper assists with this process.  The argument list is as followed:

```javascript
TestService.result(
	'The array of results you want to package.',
	function(err, result) {
		// The callback to fire once the Result package is created.
		if(err) throw err;
		console.log(result.response());
	}, 
	'Criteria you used to get this result',
	'An array of custom messages or warnings associated with this result',
	'Errors get added to this parameters'
);
```

Generally, your methods will return a positive result, which would look like:

```javascript
TestService.result(
	'2',
	done, 
	'1+1=?'
);
```

Occasionally messages or additional metadata needs to be attached to the response:

```javascript
TestService.result(
	'2',
	done, 
	'1+1=?',
	['Metadata note: I love doing math :)']
);
```

And regarding errors, pass them into the fifth argument:

```javascript
// An example of a response package with an error
TestService.result(
	'3',
	done, 
	'1+1=?',
	null,
	'Invalid mathing'
);
```
