# Express Example

This repository demonstrates the usage of Sequelize within an [Express](https://expressjs.com) application and SQL Server in the backend.
The implemented logic is a simple task tracking tool.


## Setup in Details

In order to understand how this application has been built, you can find the
executed steps in the following snippet. You should be able to adjust those
steps according to your needs. Please note that the view and the routes aren't
described. You can find those files in the repo.

#### Express Setup

First we will create a bare Express App using `express-generator` [Express Generator](https://expressjs.com/en/starter/generator.html)
```bash
# install express generator globally
npm install -g express-generator

# create the sample app
mkdir express-example
cd express-example
express -f

# install all node modules
npm install
```

#### Sequelize Setup

Now we will install all sequelize related modules.

```bash
# install ORM , CLI and SQLite dialect
npm install --save sequelize sequelize-cli sqlite3

# generate models
node_modules/.bin/sequelize init
node_modules/.bin/sequelize model:create --name User --attributes username:string
node_modules/.bin/sequelize model:create --name Task --attributes title:string
```

You will now have a basic express application with some additional directories
(config, models, migrations). Also you will find two migrations and models.
One for the `User` and one for the `Task`.

In order to associate the models with each other, you need to change the models
like this:

```js
// task.js
// ...
classMethods: {
  associate: function(models) {
    Task.belongsTo(models.User);
  }
}
// ...
```

```js
// user.js
// ...
classMethods: {
  associate: function(models) {
    User.hasMany(models.Task)
  }
}
// ...
```

If you want to use the automatic table creation that sequelize provides,
you have to adjust the `bin/www` file to this:

```js
#!/usr/bin/env node

var app = require('../app');
var debug = require('debug')('init:server');
var http = require('http');
var models = require("../models");

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

var server = http.createServer(app);

// sync() will create all table if they doesn't exist in database
models.sequelize.sync().then(function () {
  server.listen(port);
  server.on('error', onError);
  server.on('listening', onListening);
});

function normalizePort(val) { /* ... */ }
function onError(error) { /* ... */ }
function onListening() { /* ... */ }
```

## Database Configuration

And finally you have to adjust the `config/config.json` to fit your environment.

In this app we will be using SQL Server database. Update config.json to include:

```js
// config.json
// ...
  "sqldemo": {
    "username": "login",
    "password": "password",
    "database": "TasklistDB",
    "host": "127.0.0.1",
    "dialect": "mssql"
}
```

You can also preselect the environment. Before running your app, you can do this in console,
```js
export NODE_ENV=sqldemo
```
Or if you are in windows you could try this:
```js
SET NODE_ENV=sqldemo
```

And here's where the connection parameters are read from config.json depending on the environment set:

```js
// index.js
// ...
var env       = process.env.NODE_ENV || "development";

var config    = require(path.join(__dirname, '..', 'config', 'config.json'))[env];
if (process.env.DATABASE_URL) {
  var sequelize = new Sequelize(process.env.DATABASE_URL);
} else {
  var sequelize = new Sequelize(config.database, config.username, config.password, {
		dialect: config.dialect,
		host: config.host,
		dialectOptions: {
			encrypt: true // required for Azure SQL Database
		}		
	});
}
// ...
```
Once thats done, your database configuration is ready!

## Starting App

```bash
npm install
npm start
```

This will start the application and create all the schema objects SQL Server database required by your app.
Just open [http://localhost:3000](http://localhost:3000).

## Running Tests

We have added some [Mocha](https://mochajs.org) based test. You can run them by `npm test` 
