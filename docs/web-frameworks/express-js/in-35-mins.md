# Learn Express JS in 35 Minutes

!!! Note

    The following content is based on the YouTube video [Learn Express JS In 35 Minutes](https://www.youtube.com/watch?v=SccSCuHhOw0).

## Project Setup

Initialize the project by generating a `package.json` file with default configurations:
``` bash
npm init -y

Wrote to /<redacted>/express-crash-course/package.json:

{
  "name": "express-crash-course",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs"
}
```

Install **Express**, the primary framework for our application:
``` bash
npm i express

added 65 packages, and audited 66 packages in 2s
```

The installation adds Express to the `dependencies` section of the `package.json` file:
``` json title="package.json"
{
...
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "express": "^5.2.1"
  }
}
```

Next, install **Nodemon** as a development dependency:
``` bash
npm i --save-dev nodemon

added 26 packages, and audited 92 packages in 8s
```

Nodemon is a utility that monitors for any changes in your source and automatically restarts your server. Configure a custom script in `package.json` to facilitate this:
``` json hl_lines="2"
  "scripts": {
    "devStart": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

Create a `server.js` file in the root directory. The resulting project structure should appear as follows:
``` bash
tree -L 1
.
├── node_modules
├── package-lock.json
├── package.json
└── server.js
```

Executing `npm run devStart` will now run `server.js` through Nodemon, based on the configuration defined in `package.json`.

---
## Server Setup

``` js title="server.js"
const express = require('express')
const app = express()

app.listen(3000)
```

- **`require('express')`**: Imports the Express library into your project.
- **`express()`**: Creates an instance of an Express application, which we've named `app`.
- **`app.listen(3000)`**: Tells the server to start listening for incoming requests on port `3000`.


---
## Basic Routing & Rendering HTML

To render HTML templates, install the **EJS (Embedded JavaScript)** view engine:
``` bash
npm i ejs

added 2 packages, and audited 94 packages in 784ms
```

Configure Express to use EJS and define a basic route:
``` js hl_lines="4-9" title="server.js"
const express = require('express')
const app = express()

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
  console.log("Here")
  res.render('index', {text: "World"})
})

app.listen(3000)
```

- **`app.set('view engine', 'ejs')`**: Configures the application to use EJS as the template engine for rendering views.
- **`app.get('/', ...)`**: Defines a route handler for GET requests to the root URL (`/`).
- **`res.render('index', {text: "World"})`**: Renders the `index.ejs` file located in the `views` directory and passes a data object (`{text: "World"}`) to the template.

Create the template file to display dynamic content:
``` html hl_lines="9" title="views/index.ejs"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  Hello <%= locals.text || 'Default' %>
</body>
</html>
```

- **`<%= locals.text %>`**: EJS syntax used to output the value of the `text` variable passed from the server into the HTML.

![hello-world](../../assets/img/web-frameworks/express-js/in-35-mins/hello-world.png)

When you navigate to `http://localhost:3000`, the server renders the `index.ejs` template, resulting in "Hello World" being displayed in the browser as shown above.

---
## Routers

To keep your application organized, you can extract related routes into separate files using Express Routers.

``` js hl_lines="11 13" title="server.js"
const express = require('express')
const app = express()

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
  console.log("Here")
  res.render('index', {text: "World"})
})

const userRouter = require('./routes/users')

app.use('/users', userRouter)

app.listen(3000)
```

- **`require('./routes/users')`**: Imports the router module defined in the `routes/users.js` file.
- **`app.use('/users', userRouter)`**: Mounts the `userRouter` middleware at the `/users` path. This means any request starting with `/users` will be handled by this router.

Create the router file to handle user-specific routes:

``` js title="routes/users.js"
const express = require('express')
const router = express.Router()

router.get('/', (req, res) => {
  res.send("User List")
})

router.get('/new', (req, res) => {
  res.send('User New Form')
})

module.exports = router
```

- **`express.Router()`**: Creates a new, isolated router object. You can define routes on this object just like you do on the main `app`.
- **`router.get('/', ...)`**: Defines a route for the root of this router (which resolves to `/users` based on how it's mounted in `server.js`).
- **`router.get('/new', ...)`**: Defines a route for `/new` relative to this router (resolves to `/users/new`).
- **`module.exports = router`**: Exports the router object so it can be imported and used in other files, like our `server.js`.

![users-new](../../assets/img/web-frameworks/express-js/in-35-mins/users-new.png)

When navigating to `http://localhost:3000/users/new`, the application uses the `userRouter` to match the `/new` path and responds with the "User New Form" text.

---
## Advanced Routing



---
## Middleware



---
## Rendering Static Files



---
## Parsing Form/JSON Data



---
## Parse Query Params
