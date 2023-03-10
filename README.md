This file may be outdated - please refer to the Extension page in VSCode or here https://marketplace.visualstudio.com/items?itemName=Alturos.vscjsontableeditor for the most current version of the documentation!

# JSON Table Editor

With this extension you can edit certain types of JSON files conveniently in a table view. It works best with arrays of objects inside of a specific wrapper structure. This editor is mainly geared towards editing testcase files for data driven testing.

![Demo](images/default-animation-2.gif?raw=true)
See below for example cats.ts.js

## Features

The editor supports different modes. These modes are set within the JSON data `{ "mode": "testcase", ... }` and each mode may have slightly different editor behaviour.

### Schema Support

The editor supports very primitive schemas for the data, the following schema pretty much has all features present:

```
...
	"schema": {
		"name": "string",
		"member": "boolean",
		"age": "number",
		"nicknames": [ "string" ],
		"cars": [{
			"brand": "string",
			"year": "number"
			"damages": [{
				"name": "string",
				"date": "string"
			}],
		}],
		"insurances": [{
			"name": "string",
			"startDate": "date",
			"endDate": "date",
		}],
	},
...
```

* String, Number, Boolean, Date
* objeces inside of objects
* arrays of objects
* arrays of primitives
* nesting

FUTURE

* Date
* expressions
	* ${ date(+1) }

### Mode `default` for `*.json` files

Even though this mode is for `*.json` files and it does not come with a base schema or any specific formatting, it requires the json files to be of a specific structure:

```
{
	"mode": "default",
	"schema": {
		"url": "string"
	},
	"rows": [
		{
			"url": "http://alturos.com"
		}
	]
}
```

### Mode `guessed` for array-of-objects files

When the file contains an array of objects, the editor will guess a schema based on the encountered data and open the file using that schema:

```
[
    {
        "name": "Google",
        "enabled": true,
        "metadata": {
            "customer": "customer1",
            "environment": "stage",
        },
        "url": "https://google.com",
        "text": "Google"
    },
    {
        "name": "Other",
        "enabled": true,
        "metadata": {
            "customer": "customer2",
            "environment": "stage",
        },
        "url": "https://other.com",
        "text": "xxx"
    }
]
```

For the file above, the editor will guess this schema:

```
{
	"name": "string",
	"enabled": "string",
	"metadata": {
		"customer": "string",
		"environment": "string",
	},
	"url": "string",
	"text": "string"
}
```

Even though this works quite well if your file has a simple schema, you should consider providing a schema manually as it gives you more control over it (see default and testcase modes).

### Mode `testcase` for `*.tc.json` files

testcase mode has base schema as follows; the metadata columns are formatted as abbreviations to save space in the table view:

```
{
	"name": "string",
	"enabled": "boolean",
	"metadata": {
		"customer": "string",
		"environment": "string",
		"email: string",
		"priority": "number",
	},
}
```

The schema specified in your file will be merged over this base schema.

In this mode the data is being stored as `{ ... "testcases": [ ... ] }` in the file.

An example file may look like this:

```
{
	"mode": "testcase",
	"command": "npm run my-tests",
	"schema": {
		"url": "string"
	},
	"testcases": [
		{
			"name": "My Case 1",
			"enabled": true,
			"metadata": {
				"customer": "C1",
				"environment": "live",
				"email": "so@alturos.com",
				"priority": 1
			},
			"url": "http://alturos.com"
		}
	]
}
```

#### Running a testcase

Next to each testcase we show a run icon. Clicking it (or pressing the hotkey CTRL-S) will execute the command specified in the file as `{ ... "command": "...", ...}`. This execution will have some useful environment vairables present:

* `$RAND` contains a random number useful for identifying result data
* `$TEST_GUESS_CYPRESS` contains a guess of the cypress spec path (we just replace '.tc.json' with '.cy.js') - this is useful for using `cypress run --spec $TEST_GUESS_CYPRESS`
* `$TESTCASE` contains the name of the testcase. We recommend implementing a filter on your testcase loading that you can trigger like `cypress run --env filterName=$TESTCASE` so you can really only run this one testcase.

You may set a very specific `command` to run your test cases, however we recommennd to only have a very minimal command like `npm run test` in your `.tc.json` files and handling the complexities in that test script or even in the test implementation.

#### ElasticSearch integration

We like reporting our test executions into an ElasticSearch. You will have to implement this reporting inside your test cases. The JSON Table Editor can show the latest reports and their fail/success status, as long as that index is compatible to what it expects. We report our test runs with code similar to this:

```
...
	const report = {
		'@timestamp': new Date().toISOString()
		test: current.spec.fileName,
		testcase: current.testcase.name,
		status,
	}
	const result = await axios.post('https://todo-your-elastic-search.com/es/testreport/_doc/', report, {
		headers: {
			'content-type': 'application/json; charset=UTF-8',
		},
		auth: {
			username: 'TODO:UUU',
			password: 'TODO:PPP'
		},
	})
...
```

To connect JSON Table Editor to your ElasticSearch, you have to set the url and auth in the VSCode extension configuration:

* vscjsontableeditor.elasticSearch.url
* vscjsontableeditor.elasticSearch.basicAuth

It can then retrieve the latest execution data per testcase. It loads `@timestamp` and `status` and filters by `test` and `testcase`, so these fields must be present.

### Hotkeys

The editor is optimized for keyboard support, mouse support is very limited, so we recomment you memorize these shortcuts to efficiently interact with your documents:

* Arrow keys - navigate cells
* OPTION-ArrowUp - move selected line up
* OPTION-ArrowDown - move selected line down
* CTRL-A - navigate to leftmost cell
* CTRL-E - navigate to rightmost cell
* Enter - edit cell / commit cell edit
* Space - toggle Boolean type cell
* CTRL-D - duplicate focussed item
* CTRL-I - insert after focussed item
* CTRL-X - delete focussed item
* CTRL-S - run focussed test case
* CMD-S - save file

## Extension Settings

The extension currently does not offer any general settings, however certain use cases do, see:

- Features > ElasticSearch Integration

## Limitations & Known Issues
- when an array is undefined, we can currently not focus on it with the keyboard
- schema depth should be below 5, otherwise you may experience problems with keyboard navigation
- the extension was only tested on Mac

## Future Features
- checkbox UI for Boolean type
- hotkey help similar to empty vsc page below table
- automatically refresh status after run testcase
- filtering
- magic default for command
- more modes - any ideas?

## Release Notes
See CHANGELOG.md for details.
### 0.7.0
- 👾 celebrating 256 installs!
- context highlight
- mouse controls (move up, move down, delete, insert, clone)
### 0.6.0
- support for `"date"` schema types
- display key-helpers (hold Ctrl or Alt/Option)
### 0.5.0
- support for arrays of primitives
### 0.4.0
- move line up/down
### 0.3.0
- edit improvements
### 0.2.0
- autogenerated schema, flat array files
### 0.1.0
- basic readyness for use with mode testcase
### 0.0.1
- Initial release of JSON Table Editor.

## Examples
### `cats.ts.js`
```
{
	"schema": {
		"name": "string",
		"age": "number",
		"address": {
			"street": "string",
			"number": "number"
		},
		"cats": [{
				"name": "string",
				"age": "number",
				"toys": [{
					"type": "string"
				}]
		}]
	},
	"rows": [
		{
			"name": "Mike",
			"age": 63,
			"address": {
				"street": "Mill Street",
				"number": 111
			},
			"cats": [
				{
					"name": "Mia",
					"age": 4,
					"toys": [ { "type": "car" }, { "type": "ball" } ]
				},
				{
					"name": "Moe",
					"age": 2,
					"toys": [ { "type": "ball" } ]
				}
			]
		},
		{
			"name": "Rose",
			"age": 61,
			"address": {
				"street": "Red Avenue",
				"number": 22
			},
			"cats": [
				{
					"name": "Randy",
					"age": 3,
					"toys": [ { "type": "mouse" }, { "type": "twine" } ]
				}
			]
		},
		{
			"name": "Lucy",
			"age": 5,
			"address": {
				"street": "Red Avenue",
				"number": 22
			},
			"cats": [
				{
					"name": "Leo",
					"age": 1,
					"toys": [ { "type": "laser" } ]
				}
			]
		}
	]
}
```
