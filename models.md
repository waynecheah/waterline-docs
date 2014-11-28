## Models

Models represent a structure of data which requires persistant storage. The data may live in any
data-store but is interfaced in the same way. This allows your users to live in PostgreSQL and your
user preferences to live in MongoDB and you will interact with the data models in the exact same way.

If you're using MySQL, a model might correspond to a table. If you're using MongoDB, it might
correspond to a collection. In either case, our goal is to provide a simple, modular way of managing
data without relying on any one type of database.

### How Do I Define A Model

Model definitions contain `attributes`, `validations`, `instance methods`, `lifecycle callbacks`
and `class methods`. To define a model you will extend the `Waterline.Collection` object and add
in your own `attributes` and methods.

By default an attribute named `id` will be automatically added to your model which will contain
an auto-incrementing number unique to each record. This will be your model's `primary key` and
will be indexed when available. You can override this if you would like to define your own primary
key factory or attribute.

Each model will also get two timestamp attributes added by default: `createdAt` and `updatedAt` which
will track when a record went into the datastore and when it was last updated.

```javascript
var Person = Waterline.Collection.extend({

  // Idenitity is a unique name for this model
  identity: 'person',

  // Connection
  // A named connection which will be used to read/write to the datastore
  connection: 'local-postgresql',

  // Attributes are basic pieces of information about a model
  attributes: {
    firstName: 'string',
    lastName: 'string',
    age: 'integer',
    birthDate: 'date',
    emailAddress: 'email'
  }
});

module.exports = Person;
```

You can also set options for each attribute. These include `validations` and any indexing or unique
properties.

```javascript
var Person = Waterline.Collection.extend({

  identity: 'person',
  connection: 'local-postgresql',

  attributes: {

    // Don't allow two objects with the same value
    lastName: {
      type: 'string',
      unique: true
    },

    // Ensure a value is set
    age: {
      type: 'integer',
      required: true
    },

    // Set a default value if no value is set
    phoneNumber: {
      type: 'string',
      defaultsTo: '111-222-3333'
    },

    // Create an auto-incrementing value (not supported by all data-stores)
    incrementMe: {
      type: 'integer',
      autoIncrement: true
    },

    // Index a value for faster queries
    emailAddress: {
      type: 'email', // Email type will get validated by the ORM
      index: true
    }
  }
});
```

### Data Types and Attribute Properties

The following data types are currently available:

* string
* text
* integer
* float
* date
* time
* datetime
* boolean
* binary
* array
* json

These will map to the underlying database type if available. If a database doesn't support a type
a polyfill will be used. For example when using an array or json type in MySQL the values will be
stringified before being saved.

#### Attribute Properties

These properties are also available on an attribute and can be used to enforce various constraints
on the data.

###### defaultsTo

Will set a default value on an attribute if one is not supplied when the record is created.

```javascript
attributes: {
  phoneNumber: {
    type: 'string',
    defaultsTo: '111-222-3333'
  }
}
```

###### autoIncrement

Will create a new auto-incrementing attribute. These should always be of type `integer` and will
not be supported in all datastores. For example MySQL will not allow more than one auto-incrementing
column per table.

```javascript
attributes: {
  placeInLine: {
    type: 'integer',
    autoIncrement: true
  }
}
```

###### unique

Ensures no two records will be allowed with the same value. This is a database level constraint so
in most cases a unique index will be created in the underlying data-store.

```javascript
attributes: {
  username: {
    type: 'string',
    unique: true
  }
}
```

###### index

Will create a simple index in the underlying datastore for faster queries if available. This is only
for simple indexes and currently doens't support compound indexes. For these you will need to create
them yourself or use a migration.

There is currently an issue with adding indexes to string fields. Because Waterline performs it's
queries in a case insensitive manner we are unable to use the index on a string attribute. There are
some workarounds being discussed but nothing is implemented so far. This will be updated in the
near future to fully support indexes on strings.

```javascript
attributes: {
  email: {
    type: 'string',
    index: true
  }
}
```

###### primaryKey

Will set the primary key of the record. This should be used when `autoPK` is set to false.

```javascript
attributes: {
  uuid: {
    type: 'string',
    primaryKey: true,
    required: true
  }
}
```

###### enum

A special validation property which will only allow values which match a whitelisted set of values.

```javascript
attributes: {
  state: {
    type: 'string',
    enum: ['pending', 'approved', 'denied']
  }
}
```

###### size

If supported in the datastore, can be used to define the size of the attribute. For example in MySQL
size can be used with a string to create a column with data type: `varchar(n)`.

```javascript
attributes: {
  name: {
    type: 'string',
    size: 24
  }
}
```

###### columnName

Override the attribute name before sending to a datastore. This allows you to have a different
interface for interacting with your data at the application layer and the data layer. It comes in
handy when integrating with legacy databases. You can have a nice API for your data and still allow
the data to be saved in legacy columns.

```javascript
attributes: {
  name: {
    type: 'string',
    columnName: 'legacy_data_user_name'
  }
}
```

### Validations

Validations are handled by [Anchor](https://github.com/balderdashy/anchor) which is based off of
[Node Validate](https://github.com/chriso/validator.js) and supports most of the properties in
node-validate. For a full list of validations see: [Anchor Validations](https://github.com/balderdashy/anchor/blob/master/lib/match/rules.js).

Validations are defined directly in your Collection attributes. In addition you may set the attribute
type to any supported Anchor type and Waterline will build a validation and set the schema type as
a string for that attribute.

Validation rules may be defined as simple values or functions (both sync and async) that return the
value to test against.

Available validations are:

```javascript
attributes: {
  foo: {
    empty: true,
    required: true,
    notEmpty: true,
    undefined: true,
    string:
    alpha: true,
    numeric: true,
    alphanumeric: true,
    email: true,
    url: true,
    urlish: true,
    ip: true,
    ipv4: true,
    ipv6: true,
    creditcard: true,
    uuid: true,
    uuidv3: true,
    uuidv4: true,
    int: true,
    integer: true,
    number: true,
    finite: true,
    decimal: true,
    float: true,
    falsey: true,
    truthy: true,
    null: true,
    notNull: true,
    boolean: true,
    array: true,
    date: true,
    hexadecimal: true,
    hexColor: true,
    lowercase: true,
    uppercase: true,
    after: '12/12/2001',
    before: '12/12/2001',
    is: /ab+c/,
    regex: /ab+c/,
    not: /ab+c/,
    notRegex: /ab+c/,
    equals: 45,
    contains: 'foobar',
    notContains: 'foobar',
    len: 35,
    in: ['foo', 'bar'],
    notIn: ['foo', 'bar'],
    max: 24,
    min: 4,
    minLength: 4,
    maxLength: 24
  }
}
```

Validations can also be defined as functions, either sync or async.

```javascript
attributes: {
  website: {
    type: 'string',
    contains: function(cb) {
      setTimeout(function() {
        cb('http://');
      }, 1);
    }
  }
}
```

Validations can also be used against other attributes using the `this` context.

```javascript
attributes: {
  startDate: {
    type: 'date',
    before: function() {
      return this.endDate;
    }
  },

  endDate: {
    type: 'date',
    after: function() {
      return this.startDate;
    }
  }
}
```

###### Validation Rules

| Name of validator | What does it check? | Notes on usage |
|-------------------|---------------------|----------------|
|after| check if `string` date in this record is after the specified `Date` | must be valid javascript `Date` |
|alpha| check if `string` in this record contains only letters (a-zA-Z) | |
|alphadashed|| does this `string` contain only numbers and/or dashes? |
|alphanumeric| check if `string` in this record contains only letters and numbers. | |
|alphanumericdashed| does this `string` contain only numbers and/or letters and/or dashes? | |
|array| is this a valid javascript `array` object? | strings formatted as arrays won't pass |
|before| check if `string` in this record is a date that's before the specified date | |
|binary| is this binary data? | If it's a string, it will always pass |
|boolean| is this a valid javascript `boolean` ? | `string`s will fail |
|contains| check if `string` in this record contains the seed | |
|creditcard| check if `string` in this record is a credit card | |
|date| check if `string` in this record is a date | takes both strings and javascript |
|datetime| check if `string` in this record looks like a javascript `datetime`| |
|decimal| | contains a decimal or is less than 1?|
|email| check if `string` in this record looks like an email address | |
|empty| Arrays, strings, or arguments objects with a length of 0 and objects with no own enumerable properties are considered "empty" | lo-dash _.isEmpty() |
|equals| check if `string` in this record is equal to the specified value | `===` ! They must match in both value and type |
|falsey| Would a Javascript engine register a value of `false` on this? | |
|finite| Checks if given value is, or can be coerced to, a finite number | This is not the same as native isFinite which will return true for booleans and empty strings |
|float| check if `string` in this record is of the number type float | |
|hexadecimal| check if `string` in this record is a hexadecimal number | |
|hexColor| check if `string` in this record is a hexadecimal color | |
|in| check if `string` in this record is in the specified array of allowed `string` values | |
|int|check if `string` in this record is an integer | |
|integer| same as above | Im not sure why there are two of these. |
|ip| check if `string` in this record is a valid IP (v4 or v6) | |
|ipv4| check if `string` in this record is a valid IP v4 | |
|ipv6| check if `string` in this record is aa valid IP v6 | |
|is| | something to do with REGEX|
|json| does a try&catch to check for valid JSON. | |
|len| is `integer` > param1 && < param2 | Where are params defined? |
|lowercase| is this string in all lowercase? | |
|max| | |
|maxLength| is `integer` > 0 && < param2 |  |
|min| | |
|minLength| | |
|not| | Something about regexes |
|notContains| | |
|notEmpty| | WTF |
|notIn| does the value of this model attribute exist inside of the defined validator value (of the same type) | Takes strings and arrays |
|notNull| does this not have a value of `null` ? | |
|notRegex| | |
|null| check if `string` in this record is null | |
|number| is this a number? | NaN is considered a number |
|numeric| checks if `string` in this record contains only numbers | |
|object| checks if this attribute is the language type of Object | Passes for arrays, functions, objects, regexes, new Number(0), and new String('') ! |
|regex| | |
|required| Must this model attribute contain valid data before a new record can be created? | |
|string| is this a `string` ?| |
|text| okay, well is <i>this</i> a `string` ?| |
|truthy| Would a Javascript engine register a value of `false` on this? | |
|undefined| Would a javascript engine register this thing as have the value 'undefined' ? | |
|uppercase| checks if `string` in this record is uppercase | |
|url| checks if `string` in this record is a URL | |
|urlish| Does the `string` in this record contain something that looks like a route, ending with a file extension? | /^\s([^\/]+\.)+.+\s*$/g |
|uuid| checks if `string` in this record is a UUID (v3, v4, or v5) | |
|uuidv3| checks if `string` in this record is a UUID (v3) | |
|uuidv4| checks if `string` in this record is a UUID (v4) | |

#### Custom Validations

You can define your own types and their validation with the `types` object. It's possible to access
and compare values to other attributes. This allows you to move validation business logic into your
models and out of your controller logic.

```javascript
var User = Waterline.Collection.extend({
  types: {
    point: function(latlng){
      return latlng.x && latlng.y
    },

    password: function(password) {
      return password === this.passwordConfirmation;
    });
  },

  attributes: {
    firstName: {
      type: 'string',
      required: true,
      minLength: 5,
      maxLength: 15
    },

    location: {
      type: 'json',
      point: true
    },

    password: {
      type: 'string',
      password: true
    },

    passwordConfirmation: {
      type: 'string'
    }
  }
});
```

### Instance Methods

You can attach instance methods to a model which will be available on any record returned from a
query. These are defined as functions in your model attributes.

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    firstName: 'string',
    lastName: 'string',
    fullName: function() {
      return this.firstName + ' ' + this.lastName;
    }
  }
});
```

#### toObject/toJSON Instance Methods

The `toObject()` method will return the currently set model values only, without any of the instance
methods attached. Useful if you want to change or remove values before sending to the client.

However we provide an even easier way to filter values before returning to the client by allowing
you to override the toJSON() method in your model.

Example of filtering a password in your model definition:

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    name: 'string',
    password: 'string',

    // Override toJSON instance method to remove password value
    toJSON: function() {
      var obj = this.toObject();
      delete obj.password;
      return obj;
    }
  }
});
```

### Lifecycle Callbacks

Lifecycle callbacks are functions you can define to run at certain times in a query. They are hooks
that you can tap into in order to change data. An example use case would be automatically encrypting
a password before creating or automatically generating a slugified url attribute.

### Callbacks on `create`

  - beforeValidate: fn(values, cb)
  - afterValidate: fn(values, cb)
  - beforeCreate: fn(values, cb)
  - afterCreate: fn(newlyInsertedRecord, cb)

#### Example

If you want to encrypt a password before saving in the database you can use the `beforeCreate`
lifecycle callback.

```javascript
var bcrypt = require('bcrypt');

var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {

    username: {
      type: 'string',
      required: true
    },

    password: {
      type: 'string',
      minLength: 6,
      required: true,
      columnName: 'encrypted_password'
    }

  },


  // Lifecycle Callbacks
  beforeCreate: function(values, next) {
    bcrypt.hash(values.password, 10, function(err, hash) {
      if(err) return next(err);
      values.password = hash;
      next();
    });
  }
});
```

### Callbacks on `update`

  - beforeValidate: fn(valuesToUpdate, cb)
  - afterValidate: fn(valuesToUpdate, cb)
  - beforeUpdate: fn(valuesToUpdate, cb)
  - afterUpdate: fn(updatedRecord, cb)

#### Example

You're the NSA and you need to update the record of a person who is a suspect!  First though, you
need to make sure that the record concerns a person of interest. You might want to use the
`beforeValidation` lifecycle callback to see if the record's `citizen_id` exists in your
`Probable_suspects` model.

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    citizen_name: 'string',
    phone_records: 'array',
    text_messages: 'array',
    friends_and_family: 'array',
    geo_location: 'json',
    loveint_rating: 'integer',
    citizen_id: 'integer'
  },

  beforeValidate: function(citizen_record, next){
    Probable_suspects.findOne(citizen_record.citizen_id).exec(function(err, suspect) {
      if(err) return next(err);
      if(!suspect) return next(new Error('This citizen is not a suspect'));
      next();
    });
  }
};
```

### Callbacks on `destroy`

  - beforeDestroy: fn(criteria, cb)
  - afterDestroy: fn(deletedRecord, cb)

#### Example

You want to update a cache to remove a record after it has been destroyed. To do this you can use
the `afterDestroy` lifecycle callback.

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    name: 'string'
  },

  afterDestroy: function(deleted_record, next){
    Cache.sync(next);
  }

};
```

### Associations

Associations are available in Waterline starting in version `0.10`. See [Associations](associations.md) for
information on how to define and query relations between your models.

### Configuration

You can define certain top level properties on a per model basis. These will define how your schema
is synced with the datastore and allows you to turn off default behaviour.

###### identity

A required property on each model which describes the name of the model. This must be unique per
instance of Waterline.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo'

});
```

###### connection

A required property on each model that describes which connection queries will be run on. You can use
either a string or an array for the value of this property. If an array is used your model will have
access to methods defined on both adapters in the connections. They will inherit from right to left
giving the adapter from the first connection priority in adapter methods.

So for example if you defined connections using both `sails-postgresql` and `sails-mandrill` and the
`sails-mandrill` adapter exposes a `send` method your model will contain all the CRUD methods exposed
from `sails-postgresql` as well as a `send` method which will be run on the mandrill adapter.

```javascript
// String Format
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql'

});

// Array Format
var Bar = Waterline.Collection.extend({

  identity: 'bar',
  connection: ['my-local-postgresql', 'sails-mandrill']

});
```

###### autoPK

A flag to toggle the automatic primary key generation. If turned off no primary key will be created
by default and one will need to be defined.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoPK: false

});
```

###### autoCreatedAt

A flag to toggle the automatic timestamp for createdAt.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoCreatedAt: false

});
```

###### autoUpdatedAt

A flag to toggle the automatic timestamp for updatedAt.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoUpdatedAt: false

});
```

###### schema

A flag to toggle schemaless or schema mode in databases that support schemaless data structures. If
turned off this will allow you to store arbitrary data in a record. If turned on, only attributes
defined in the model's attributes object will be allowed to be stored.

For adapters that don't require a schema such as Mongo or Redis the default setting is to be
schemaless.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  schema: true

});
```

###### tableName

You can define a custom table name on your adapter by adding a `tableName` attribute. If no table
name is supplied it will use the identity as the table naafterDestroyme when passing it to an adapter.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  tableName: 'my-legacy-table-name'

});
```

### Class Methods

"Class" methods are functions available at the top level of a model. They can be called anytime after
a Waterline instance has been initialized.

These are useful if you would like to keep model logic in the model and have reusuable functions
available.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  attributes: {},

  // A "class" method
  method1: function() {}

});

// Example
Foo.method1()
```
