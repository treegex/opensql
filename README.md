# OpenSql

## Table of Contents

- [Install](#install)
- [Introduction](#introduction)
- [Establishing connections](#establishing-connections)
- [Connection options](#connection-options)
- [Type casting](#type-casting)
    - [Number](#number)
    - [Date](#date)
    - [String](#string)
    - [Spatial](#spatial)
- [Functions](#functions)
    - [connect](#connect)
    - [createDatabase](#createDatabase)
    - [dropTable](#dropTable)
    - [addForeignKey](#addForeignKey)
    - [remove](#remove)
    - [update](#update)
    - [addMultiValue](#addMultiValue)
    - [addOne](#addOne)
    - [addWithFind](#addWithFind)
    - [customQuery](#customQuery)
    - [find](#find)
    - [findTable](#findTable)
    - [result](#result)
- [Sql injection](#sql-injection)
- [License](#license)

## Install

This is a [Node.js](https://nodejs.org/en/) module available through the
[npm registry](https://www.npmjs.com/).

Before installing, [download and install Node.js](https://nodejs.org/en/download/). Node.js 0.6 or higher is required.

Installation is done using the
[`npm install` command](https://docs.npmjs.com/getting-started/installing-npm-packages-locally):

```sh
$ npm install opensql
```

## Introduction

OpenSql is a promise-based Node.js ORM tool for MySql.

Here is an example on how to use it:

```js
let openSql = require('opensql');

openSql.connect({
    host: 'localhost',
    user: 'me',
    password: 'secret',
    database: 'my_db'
});

openSql.customQuery('SELECT 1 + 1 AS solution')
    .result(result => {
        console.log(result);
    });
```

## Establishing connections

The recommended way to establish a connection is this:

```js
let openSql = require('opensql');

openSql.connect({
    host: 'localhost',
    user: 'me',
    password: 'secret',
    database: 'my_db'
}).result(result => {
    console.log(result);
});
```

## Connection options

OpenSql allows you to use all parts of the connection options

See this [link](https://github.com/mysqljs/mysql#connection-options) for more information on connection options

## Type casting

### Number

* BIT
* INT
* REAL
* FLOAT
* BIGINT
* SERIAL
* DOUBLE
* DECIMAL
* TINYINT
* BOOLEAN
* SMALLINT
* MEDIUMINT

### Date

* TIME
* YEAR
* DATE
* DATETIME
* TIMESTAMP

### String

* SET
* CHAR
* TEXT
* ENUM
* BLOB
* BINARY
* VARCHAR
* TINYBLOB
* LONGBLOB
* TINYTEXT
* LONGTEXT
* VARBINARY
* MEDIUMBLOB
* MEDIUMTEXT

### Spatial

* POINT

# Functions

## connect

The connect function uses mysql createConnection () to create the connection.

```js
// The argument ->  JsonObject 
openSql.connect();
```

See this [link](https://github.com/mysqljs/mysql#connection-options) for more information on connect options

**Example**

```js
openSql.connect({
    host: 'localhost',
    user: 'me',
    password: 'secret',
    database: 'my_db'
});
```

## createDatabase

```js
// The argument ->  String
openSql.createDatabase();
```

**Example**

```js
openSql.createDatabase('my_db');
```

## createTable

```js
// The argument ->  JsonObject
openSql.createTable();
```

**Example**

```js
let openSql = require('opensql'),
    {
        INT,
        VARCHAR,
        BOOLEAN
    } = openSql.dataType,
    {
        NOT_NULL,
        AUTO_INCREMENT
    } = openSql.sqlKeyword;

openSql.createTable({
    table: 'users',
    field: {
        id: INT([NOT_NULL, AUTO_INCREMENT]),
        phone: VARCHAR(20),
        isActive: BOOLEAN(),
        authCode: INT(6),
        name: VARCHAR(20)
    },
    primaryKey: 'id'
});
```

## dropTable

```js
// The argument ->  Array Or String
openSql.dropTable();
```

**Example**

Drop a table

```js
openSql.dropTable('my_table');
```

Drop multi tables

```js
openSql.dropTable(['users', 'books']);
```

## addForeignKey

```js
// The argument -> JsonObject
openSql.addForeignKey();
```

**Example**

```js
let openSql = require('opensql'),
    {
        CASCADE
    } = openSql.sqlKeyword;

openSql.addForeignKey({
    table: 'book',
    foreignKey: 'author',
    referenceTable: 'users',
    field: 'id',
    onDelete: CASCADE, // options -> SET_NULL | RESTRICT | NO_ACTION
    onUpdate: CASCADE  // options -> SET_NULL | RESTRICT | NO_ACTION
});
```

## remove

```js
// The argument -> JsonObject
openSql.remove();
```

Delete row in table

**Example**

```js
openSql.remove({
    table: 'users',
    where: {
        phone: '09...'
    }
});
```

**Original query**

```sql
DELETE
FROM users
WHERE phone = '09...'
```

This is a minimal example , You can write advanced sql query , Here is example:

```js
let openSql = require('opensql'),
    {
        OR
    } = openSql.queryHelper;

openSql.remove({
    table: 'users',
    where: {
        phone: '09...',
        op: [
            OR({
                isOnline: false
            })
        ]
    }
});
```

**Original query**

```sql
DELETE
FROM users
WHERE phone = '09...'
   OR isOnline = false
```

Other examples:

**Example**

```js
let openSql = require('opensql'),
    {
        OR,
        AND
    } = openSql.queryHelper;

openSql.remove({
    table: 'users',
    where: {
        phone: '09...',
        op: [
            OR({
                isOnline: false
            }, AND({
                username: 'treegex'
            }))
        ]
    }
});
```

**Original query**

```sql
DELETE
FROM users
WHERE phone = '09...'
   OR isOnline = false AND username = 'treegex'
```

**Example**

```js
let openSql = require('opensql'),
    {
        IN,
        OR,
        AND,
        LIKE,
        BETWEEN,
        setOperator
    } = openSql.queryHelper,
    keyHelper = openSql.keywordHelper;

openSql.remove({
    table: 'users',
    where: {
        id: BETWEEN(5, 10),
        age: IN([0, 50], keyHelper.OR), // default is keyHelper.AND you can change to keyHelper.OR 
        op: [
            AND({
                isOnline: false
            })
        ],
        price: setOperator(LESS_THAN, 15),
        text: LIKE('%a', keyHelper.OR)
    }
});
```

keyHelper.OR and keyHelper.AND Used when two or more conditions are used in a query, Example:

```js
let openSql = require('opensql'),
    {
        IN,
        BETWEEN
    } = openSql.queryHelper,
    keyHelper = openSql.keywordHelper;

openSql.remove({
    table: 'users',
    where: {
        id: BETWEEN(5, 10),
        age: IN([0, 50], keyHelper.OR),
        status: true
    }
});
```

**Original query**

```sql
DELETE
FROM users
WHERE id BETWEEN 5 AND 10
   OR age IN (0, 50) AND status = true
```

**BETWEEN**

```js
/**
 Arg1 =>  Int
 Arg2 =>  Int
 Arg3 =>  String -> default is  keyHelper.AND you can change to keyHelper.OR
 */

BETWEEN();
```

```js
/**
 Arg1 => Array
 Arg2 => String -> default is  keyHelper.AND you can change to keyHelper.OR
 */

IN();
```

```js
/**
 Arg1 => String
 Arg2 => String -> default is  keyHelper.AND you can change to keyHelper.OR
 */

LIKE();
```

setOperator function, Help to use another operator

```js
/**
 Arg1 => String -> Valid operator (EQUAL_TO, LESS_THAN, GREATER_THAN, NOT_EQUAL_TO, LESS_THAN_OR_EQUAL_TO, GREATER_THAN_OR_EQUAL_TO)
 Arg2 => Any
 Arg3 => String -> default is  keyHelper.AND you can change to keyHelper.OR
 */

setOperator();
```

## update

You can use all the functions of the delete function for the update function, Example:

```js
let openSql = require('opensql');

openSql.update({
    table: 'users',
    edit: {
        username: 'treegex'
    },
    where: {
        id: 5
    }
});
```

**Original query**

```sql
UPDATE users
SET username = 'treegex'
WHERE id = 5
```

## addMultiValue

OpenSql lets you write multiple queries, Example:

```js
openSql.addMultiValue({
    table: 'book',
    data: [
        'clean code', 'Robert C Martin',
        'javascript ES6', 'Mozilla',
        'object oriented programing software engineer', 'Ivar Jacobson'
    ],
    column: ['name', 'author']
});
```

**Original query**

```sql
INSERT INTO book (name, author)
VALUES ('clean code', 'Robert C Martin'),
       ('javascript ES6', 'Mozilla'),
       ('object oriented programing software engineer', 'Ivar Jacobson');
```

## addOne

Insert data into table.

```js
openSql.addOne({
    table: 'book',
    data: {
        name: 'clean code',
        author: 'Robert C Martin'
    }
});
```

**Original query**

```sql
INSERT INTO book (name, author)
VALUES ('clean code', 'Robert C Martin');
```

## addWithFind

Selects data before inserting, Example:

```js
let openSql = require('opensql'),
    keyHelper = openSql.keywordHelper;

openSql.addWithFind({
    table: 'post',
    optKey: [
        keyHelper.EQUAL_TO
    ],
    data: [
        ['name', 'img'], 'post', 'id', '5'
    ],
    where: true
});
```

**Original query**

```sql
INSERT INTO post (name, img)
SELECT name, img
FROM post
WHERE id = 5;
```

## customQuery

Use special query, Example:

```js
openSql.customQuery('SELECT 1 + 1 AS solution')
    .result(result => {
        console.log(result);
    });
```

## find

Find data in table, Example:

```js
let openSql = require('opensql'),
    keyHelper = openSql.keywordHelper;

openSql.find({
    optKey: [
        keyHelper.AND
    ],
    data: [
        'phone', 'users', 'phone', `09...`, 'status', false
    ],
    where: true
});
```

**Original query**

```sql
SELECT phone
FROM users
WHERE phone = '09...'
  AND status = false;
```

Other Examples:

**Example**

```js
let openSql = require('opensql'),
    keyHelper = openSql.keywordHelper;

openSql.find({
    optKey: [
        keyHelper.COUNT
    ],
    data: ['users']
});
```

**Original query**

```sql
SELECT COUNT(*)
FROM users;
```

**Example**

```js
let openSql = require('opensql'),
    keyHelper = openSql.keywordHelper;

openSql.find({
    optKey: [
        keyHelper.STAR
    ],
    data: ['users']
});
```

**Original query**

```sql
SELECT *
FROM users;
```

**Example**

```js
let openSql = require('opensql'),
    keyHelper = openSql.keywordHelper;

openSql.find({
  optKey: [
    keyHelper.UNION
  ],
  data: [
    'id', 'city', 'id', 'users'
  ]
});
```

**Original query**

```sql
SELECT id
FROM city UNION SELECT id FROM users;
```

## findTable

Search table in database.

```js
openSql.findTable('users');
```

**Original query**

```sql
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = users
  AND TABLE_SCHEMA = treegex;
```

## result

Returns result of query, Example:

```js
openSql.customQuery('SELECT 1 + 1 AS solution')
    .result(result => {
        console.log(result);
    });
```

## Sql injection

OpenSql has tried to make sql injection more readable and understandable to developers.

Here is an example on how to use it:

```js
let openSql = require('opensql'),
    {
        EQUAL_TO
    } = opensql.sqlKeyword;

openSql.find({
    optKey: [
        EQUAL_TO
    ],
    data: ['phone', 'users', 'phone', '09...'],
    where: true
}).result(result => {
    console.log(result);
});
```

**Original query**

```sql
SELECT phone
FROM users
WHERE phone = '09...'
```

In this example you see three parts. each part helps openSql to generate query.

## Licensing

The code in this project is licensed under the [MIT](https://github.com/treegex/opensql/blob/main/License)