AdminBro is an __automatic admin interface__, which allows you to manage your database resources. It can be added to almost any Node.js framework. In this short tutorial, I will show you how to configure AdminBro to manage _User_ [mongoosejs](https://mongoosejs.com/) resource in a simple [express.js](https://expressjs.com/) app.

We will:

1. Create a simple express app
2. Add AdminBro to it

## Let's do this!

In the first step, we will create a simple __one file__ API server.

## An example app without an AdminBro

The app is an [express.js](https://expressjs.com/) API server, which uses [mongoosejs](https://mongoosejs.com/) to fetch data from MongoDB and returns it in a JSON format.

The code of the app looks like this:

```javascript
// ==========  server.js ==============
// Requirements
const mongoose = require('mongoose')
const express = require('express')
const bodyParser = require('body-parser')

// Express server definition
const app = express()
app.use(bodyParser.json())

// Resources definitions
const User = mongoose.model('User', { name: String, email: String, surname: String })

// Routes definitions
app.get('/', (req, res) => res.send('Hello World!'))

// Route, which returns last 100 users from the database
app.get('/users', async (req, res) => {
  const users = await User.find({}).limit(10)
  res.send(users)
})

// Route, which creates new user
app.post('/users', async (req, res) => {
  const user = await new User(req.body.user).save()
  res.send(user)
})

// Running the server
const run = async () => {
  await mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true })
  await app.listen(8080, () => console.log(`Example app listening on port 8080!`))
}

run()
```

To run it you have to install dependencies:

```sh
npm install mongoose express body-parser
```

Make sure you have a [MongoDB](https://www.mongodb.com/) running in the system.

Now you can run the server via `node` command:

```sh
node server.js
```

Server exposes 2 routes: `GET /users` and `POST /users` You can test them by using [curl](https://curl.haxx.se/):

```sh
curl -d '{"user": {"name": "wojtek"}}' -H "Content-Type: application/json" -X POST http://localhost:8080/users/
# => {"_id":"5c546dcbc98f5e1923c8eef9","name":"wojtek","__v":0}

curl http://localhost:8080/users
# => [{"_id":"5c546dcbc98f5e1923c8eef9","name":"wojtek","__v":0}]
```

## Adding AdminBro

OK, so now we would like to add an __admin UI__ with the ability to manage users. Of course, we will be using __AdminBro__ in order to do that. 

First, let's install all dependencies. We are using Mongoose, hence we will have to install admin-bro-mongoose database adapter. Also we use Express.js, that is why we will need admin-bro-expressjs plugin:

```sh
npm install admin-bro admin-bro-mongoose admin-bro-expressjs
```

Now we have to use said modules:
- use {@link module:AdminBroExpressjs AdminBroExpressjs} module from 'admin-bro-expressjs' package to attach AdminBro into Express routes.
- use {@link module:AdminBroMongoose AdminBroMongoose} adapter from 'admin-bro-mongoose' package to add Mongoose resources to the admin panel. 

This is how it will look like:

```javascript
const AdminBro = require('admin-bro')
const AdminBroExpressjs = require('admin-bro-expressjs')

// We have to tell AdminBro that we will manage Mongoose resources with it
AdminBro.registerAdapter(require('admin-bro-mongoose'))

// Pass all configuration settings to AdminBro
const adminBro = new AdminBro({
  resources: [User],
  rootPath: '/admin',
})

// Build and use a router, which will handle all AdminBro routes
const router = AdminBroExpressjs.buildRouter(adminBro)
app.use(adminBro.options.rootPath, router)
```

## Full application in one file

This is an entire Express server with AdminBro managing Mongoose user resource:

```javascript
// ==========  server.js ==============
// Requirements
const mongoose = require('mongoose')
const express = require('express')
const bodyParser = require('body-parser')
const AdminBro = require('admin-bro')
const AdminBroExpressjs = require('admin-bro-expressjs')

// We have to tell AdminBro that we will manage Mongoose resources with it
AdminBro.registerAdapter(require('admin-bro-mongoose'))

// Express server definition
const app = express()
app.use(bodyParser.json())

// Resources definitions
const User = mongoose.model('User', { name: String, email: String, surname: String })

// Routes definitions
app.get('/', (req, res) => res.send('Hello World!'))

// Route, which returns last 100 users from the database
app.get('/users', async (req, res) => {
  const users = await User.find({}).limit(10)
  res.send(users)
})

// Route, which creates new user
app.post('/users', async (req, res) => {
  const user = await new User(req.body.user).save()
  res.send(user)
})

// Pass all configuration settings to AdminBro
const adminBro = new AdminBro({
  resources: [User],
  rootPath: '/admin',
})

// Build and use a router, which will handle all AdminBro routes
const router = AdminBroExpressjs.buildRouter(adminBro)
app.use(adminBro.options.rootPath, router)

// Running the server
const run = async () => {
  await mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true })
  await app.listen(8080, () => console.log(`Example app listening on port 8080!`))
}

run()
```

To run it simply write:

```sh
node server.js
```

and, under `localhost:8080/admin`, you should see something like this:

<img src="images/one-file-example.png">

## What's next

Now you know how to add AdminBro to simple Express.js app. If you are using a different framework
or you want to discover different authentication options - visit the next tutorial {@tutorial 02-frameworks}.

<a href="/tutorial-02-frameworks.html" class="button">See all the supported frameworks</a>

or skip it and go right away to:

<a href="/tutorial-03-passing-resources.html" class="button">Passing Resources</a>
