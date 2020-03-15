# Overview

RQL is the query language used in Mencius.dev to query code.

```sql
SELECT   _____
WHERE    _____
GROUP BY _____
```

## Note
v0 of RQL is very much alpha and experimental. It will be deprecated soon and replaced with something much better.


# SELECTS

```sql
SELECT A, B, C ...
```

The `SELECT` statement is a sequence of queries.

The sequencing does not work well right now. Specifically, it's not doing the cross product semantics that you would expect from SQL.

The best way to use this right now is to chain graph traversals with the javascript query

## file + exists query

file:() exists:(true)
exact_file:() exists:(true)

Checks to see if file exists on that path.


## json query

Allows you to query JSON type documents. This currently includes `.json` and `.yaml` files.

```
json:(.<key>.<key2>.<etc>......)
```

The `file:(...)` modifier can also be used on this query

```
file:() json:()
```


### Example

```json
{
  "dependencies": {
    "mylibrary": "^1.0.0"
  }
}
```

```
json:(.dependencies.mylibrary)
```

## javascript

This is the meat of RQL. Allows you to query the javascript 

### Chaining and graph traversal

```sql
SELECT
javascript:(<q1>) as one,
javascript:(<q2>) as two,
...
```


`from[...]`


### Roots

#### require

`require[...]` - Selects a require

```js
const express = require('express');
```

```sql
SELECT
javascript:(
  require[express]
)
```

#### class

`class[...]` - Selects a ClassDeclaration

```js
class InvoiceService {
  ...
}
```

```sql
SELECT
javascript:(
  class[InvoiceService]
)
```

#### string

`string[...]` - Get a string literal

```js
const blah = 'blah';
```

```sql
SELECT
javascript:(
  string[blah]
)
```

#### from

`from[...]` - Start traversing from previous nodes.

```js
const express = require('express');

const app = express();

app.use(ErrorHandler(...)); // select this call
```

```sql
SELECT
javascript:(
  require[express]
    .calls
) AS express_instances,
javascript:(
  from[express_instances]
    .member[use]
    .calls
)
```

This will select the ErrorHandler call.



### Accessors


#### instances

`instances` - Get all instances of a class

```js
class InvoiceService {
  ...
}

const invoiceService = new InvoiceService(); // select this instance
```

```sql
SELECT
javascript:(
  class[InvoiceService]
    .instances
)
```

#### calls

`calls` - Get all calls of a function

```js
const express = require('express');

const app = express(); // select this call
```

```sql
SELECT
javascript:(
  require[express]
    .calls
)
```

#### declarations

`declarations` << don't use this

#### members

`member[...]` - Get the member of a 

```js
const express = require('express');

const app = express();

app.use(ErrorHandler(...)); // select this call
```

```sql
SELECT
javascript:(
  require[express]
    .calls
    .member[use]
    .calls
)
```

`any_member` - Selects any member

```js
const express = require('express');

const app = express();

app.use(ErrorHandler(...)); // select this call

app.post('/endpoint', ...); // select this call

app.get('/endpoint', ...); // select this call
```

```sql
SELECT
javascript:(
  require[express]
    .calls
    .any_member
    .calls
)
```

#### args

`args[...]` - get an argument on a call by index

```js
const express = require('express');
const { ErrorHandler } = require('@rambutan/node-utilities');

const app = express();

app.use(
  ErrorHandler(...) // select this call on ErrorHandler
); 
```

```sql
SELECT
javascript:(
  require[express]
    .calls
    .member[use]
    .calls
    .args[0]
)
```


`is_arg` - selects all calls the item is an argument on

This will select the `app.use` call from the code above.

```sql
SELECT
javascript:(
  require[@rambutan/node-utilities]
    .member[ErrorHandler]
    .calls
    .is_arg
)
```

#### filter

`filter_type[...]`


#### follows 

`follow_exports`

`follow_require`

`follow_up_to[...]`

`follow_args[...]`


# WHERE

TBC


# GROUP BY

TBC
