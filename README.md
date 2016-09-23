# oracle-migrate
Small migrate framework for Oracle DB

## Installation

    $ npm install oracle-migrate

## Configuration

Currently you should have theese evironment variables configured in your system:

```
  database username:      NODE_ORACLEDB_USER
  database password:      NODE_ORACLEDB_PASSWORD
  url to database:        NODE_ORACLEDB_CONNECTIONSTRING
  (e.g. `localhost:1521`)
```

## Usage

```
Usage: migrate [options] [command]

Commands:

    down             migrate down by 1 file
    down   [name]    migrate down till given file name migration
    down   all       migrate down to init state
    up               migrate till most recent migration file
    up     [name]    migrate up till given migration (the default command)
    create [title]   create a new migration file with [title]

    history          fetches migration history from the database and shows it
    --install-dep    executes npm to install and save for you dependencies

    help             prints help
```

## Creating Migrations

To create a migration, execute `migrate create` with an title. This will create a node module within `./migrations/` which contains the following two exports and related sql files in the `./migrations/sql` folder:

    exports.up = function(next){
      ...
    };

    exports.down = function(next){
      ...
    };

For example:

    $ migrate create add-pets
    $ migrate create add-owners

SQL files are created empty, so you can write there your own code. Oracle database can't execute multiple SQL statements in one time, so they should be separated by delimiter:

    `-----`

For example:

```
CREATE TABLE COUNTRIES
(
  ID VARCHAR2(20) NOT NULL
, TITLE VARCHAR2(20)
, CONSTRAINT COUNTRIES_PK PRIMARY KEY
  (
    ID
  )
  ENABLE
)
-----
CREATE TABLE PLACES
(
  ID VARCHAR2(20) NOT NULL
, NAME VARCHAR2(20)
, CONSTRAINT PLACES_PK PRIMARY KEY
  (
    ID
  )
  ENABLE
)
```

These SQL statements would be executed in a sequence. When one fails - revert migration will be applied (from currently executed file - `down`).

## Running Migrations

When first running the migrations, all will be executed in sequence.

    $ migrate up
    up : migrations/1316027432511-add-pets.js
    up : migrations/1316027432512-add-jane.js
    up : migrations/1316027432575-add-owners.js
    up : migrations/1316027433425-coolest-pet.js
    migration : complete

Subsequent attempts will simply output "complete", as they have already been executed in this machine. This module knows this because it stores the current state in the database table called `MIGRATIONS`.

    $ migrate
    migration : complete

If we were to create another migration using `migrate create`, and then execute migrations again, we would execute only those not previously executed:

    $ migrate
    up : migrates/1316027433455-coolest-owner.js

You can also run migrations incrementally by specifying a migration.

    $ migrate up 1316027433425-coolest-pet.js
    up : migrations/1316027432511-add-pets.js
    up : migrations/1316027432512-add-jane.js
    up : migrations/1316027432575-add-owners.js
    up : migrations/1316027433425-coolest-pet.js
    migration : complete

This will run up-migrations upto (and including) `1316027433425-coolest-pet.js`. Similarly you can run down-migrations upto (and including) a specific migration, instead of migrating all the way down.

    $ migrate down 1316027432512-add-jane.js
    down : migrations/1316027432575-add-owners.js
    down : migrations/1316027432512-add-jane.js
    migration : complete

When you run `down all` it will revert all migrations till initial state (no migrations applied):

    $ migrate down all
    down : migrations/1316027432575-add-owners.js
    down : migrations/1316027432512-add-jane.js
    migration : complete
