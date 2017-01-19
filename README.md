# raml-validator-loader [![Velocity](http://jenkins.mesosphere.com/buildStatus/icon?job=raml-validator-loader-master)](http://jenkins.mesosphere.com/view/DCOS%20UI/job/raml-validator-loader-master/)

A webpack plugin that converts RAML rules into pure javascript-only validation routines

## Usage

```js
import TypeValidators from './ramls/myTypes.raml';

// The raml-validator-loader will convert the RAML document into an object with
// a validation function for every type defined in your RAML.

var userInput = read_user_input();
var validationErrors = TypeValidators.MyType(userInput);

// Display errors
validationErrors.forEach((error) => {
  console.log('Error message: /' + error.path.join('/'));
  console.log('  Error path : ' + error.message);
});

```

### Customizing validator

It is possible to customize the validator instance in order for example to provide custom error messages:

```js
import TypeValidators from './ramls/myTypes.raml';

/**
 * You can clone the default validator instance and customize it
 */
const CustomTypeValidators = TypeValidators.clone({
	'errorMessages': {
		// You can provide a string override
		TYPE_NOT_OBJECT: 'Expecting an object here',

		// Or a function override if you want more control over
		// the error message
		STRING_PATTERN: (templateVars, path) => {
			if (path[0] == 'name') {
				return 'Name must only contain letters';
			}

			return `Must match '${templateVars.pattern}'`;
		}
	}
});
```

## Installation

First install the module

```
npm install --save-dev raml-validator-loader
```

And then use it in your `webpack.config.js` like so:

```js
    module: {
        loaders: [
            { test: /\.raml$/, loader: "raml-validator-loader" }
        ]
    }
```

## API Reference

When you are importing a `.raml` document you will end-up loading an instance of a `RAMLValidator` class and all the RAML types are exposed as instance property functions.

```js
import TypeValidators from './ramls/myTypes.raml';
```

### `RAMLValidator` Class

A default instance of `RAMLValidator` class is returned when importing a RAML file. You don't have access to it's constructor directly.

#### Function `.clone([config])`

Creates a new instance of `RAMLValidator` allowing further customisations to the configuration. Parameters not overriden through the configuration will be kept intact.

```js
const CustomValidator = TypeValidators.clone({
	errorMessages: { ... }
});
```

##### Parameters

- **config** - [Optional] An object with the new configuration parameters to pass down to the new validator instance. Accepts the following properties:
	- `errorMessages` : An object with custom error messages. The value to the error message can either be a string, or a function with the following signature: `(messageVariables, path)` that returns an error string. For the full list of error messages you can refer to the end of this documentation.

##### Returns

- **RAMLValidator** - Returns a new `RAMLValidator` instance

#### Function `.<TypeName>(input)`

For each type in the RAML document, a validation function will be created with the above signature. You can call this function, passing it the data you want to validate and it will return an array with the errors found in it.

```js
const errors = TypeValidators.SomeRAMLType(userData);
if (errors.length > 0) {
	throw new TypeError('You must provide some valid data');
}
```

##### Parameters

- **input** - The data to validate. Can be anything

##### Returns

- **[RAMLError]** - Returns an array of `RAMLError` instances, one for every error encountered in the input data.


```js
module.exports = {

  /**
   * For every type in the RAML document, an equivalent validation function
   * will be generated by the loader.
   */
  RamlTypeName: function( validatorInput ) {

    ...

    // Each validation function will return an array of RAMLError objects
    // with the information for the validation errors occured.
    return [ RAMLError() ];
  },

}
```

### `RAMLError` Class

This class is instantiated by the type validator function and it contains the information to locate and describe an error in the type.

```js
const errors = TypeValidators.SomeRAMLType(userData);
if (errors.length > 0) {
	const error = errors[0];
	console.log('Error path = ', error.path);
	console.log('Error message = ', error.message);
}
```

#### Property `.path`

- **Array** - Returns the path in the object where the validation error ocurred, as an array of path components. For example: `['objects', 0, 'name']`

#### Property `.message`

- **String** - Returns the human-readable description of the error. For example: `Missing property 'name'`.

### Error Messages

The following error messages are used by the RAML validator. You can override them using the `.clone({errorMessages: ...})` function.

<table>
	<tr>
		<th>Constant Name</th>
		<th>Default Value</th>
	</tr>
    <tr>
        <td><code>ENUM</code></td>
        <td>Must be one of {values}</td>
    </tr>
    <tr>
        <td><code>ITEMS_MAX</code></td>
        <td>Must contain at most {value} items in the array</td>
    </tr>
    <tr>
        <td><code>ITEMS_MIN</code></td>
        <td>Must contain at least {value} items in the array</td>
    </tr>
    <tr>
        <td><code>LENGTH_MAX</code></td>
        <td>Must be at most {value} characters long</td>
    </tr>
    <tr>
        <td><code>LENGTH_MIN</code></td>
        <td>Must be at least {value} characters long</td>
    </tr>
    <tr>
        <td><code>NUMBER_MAX</code></td>
        <td>Must be smaller than or equal to {value}</td>
    </tr>
    <tr>
        <td><code>NUMBER_MIN</code></td>
        <td>Must be bigger than or equal to {value}</td>
    </tr>
    <tr>
        <td><code>NUMBER_TYPE</code></td>
        <td>Must be of type `{value}`</td>
    </tr>
    <tr>
        <td><code>PROP_MISSING</code></td>
        <td>Missing property</td>
    </tr>
    <tr>
        <td><code>PROP_MISSING_MATCH</code></td>
        <td>Missing a property that matches `{name}`</td>
    </tr>
    <tr>
        <td><code>STRING_PATTERN</code></td>
        <td>Must match the pattern "{pattern}"</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_ARRAY</code></td>
        <td>Expecting an array</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_BOOLEAN</code></td>
        <td>Expecting a boolean value</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_DATETIME</code></td>
        <td>Expecting a date/time string</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_INTEGER</code></td>
        <td>Expecting an integer number</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_NUMBER</code></td>
        <td>Expecting a number</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_OBJECT</code></td>
        <td>Expecting an object</td>
    </tr>
    <tr>
        <td><code>TYPE_NOT_STRING</code></td>
        <td>Expecting a string</td>
    </tr>
</table>

#### Function Overrides

If you want more control over the error message, you can use a function instead of string for the error message. The parameters passed to the function are the `messageVariables` that provide some contextualization to the error message, and the `path` of the error.

For example:

```js
const CustomTypeValidators = TypeValidators.clone({
	'errorMessages': {
		STRING_PATTERN: (templateVars, path) => {
			switch (path.join('.')) {
				case 'name':
					return 'Name must only contain numbers and letters'

				case 'file.name':
					return 'All file names must only contain numbers and letters'

				default:
					return `Must match ${templateVars.pattern}`;
			}
		}
	}
});
```

## Work in progress

The following **facets** are not yet implemented:

- `discriminator` in [Unions](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#using-discriminator)
- `discriminatorValue` [Unions](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#using-discriminator)

The following **types** are not yet implemented:

- [`date-only`](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#date)
- [`time-only `](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#date)
- [`datetime-only `](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#date)
- [`file`](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#file)

The following **concepts** are not yet implemented:

- [Multiple Inheritance](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#multiple-inheritance)
