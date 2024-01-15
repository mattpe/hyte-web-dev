# Getting started with Node.js & Express development

- [Getting started with Node.js](https://nodejs.dev/en/learn/) - Node.js official documentation
- [Express.js](https://expressjs.com/) - Express.js official documentation

## Setting up the first project

Prerequicities: Toolchain installed.

1. `npm init` wizard needs an interactive shell, so use VS Code terminal or Git Bash
1. Setup eslint: `npm init @eslint/config`, remember to select `node` environment
   - √ How would you like to use ESLint? _To check syntax, find problems, and enforce code style_
   - √ What type of modules does your project use? _JavaScript modules (import/export)_
   - √ Which framework does your project use? _none of these_
   - √ Does your project use TypeScript? _No_
   - √ Where does your code run? _Node_
   - √ How would you like to define a style for your project? _Use a popular style guide_
   - √ Which style guide do you want to follow? _Google_
   - √ What format do you want your config file to be in? _JavaScript_
   - √ Would you like to install them now? _Yes_
   - √ Which package manager do you want to use? _npm_
1. Install `nodemon` as a development dependency: `npm install --save-dev nodemon`
1. Review `package.json`, add a script for starting your app with nodemon and type property

   ```json
   ...
   "type": "module",
   "scripts": {
    "dev": "nodemon src/index.js",
    ...
   ```

1. Create a `.gitignore` file and add `node_modules` to it, keep it alway up to date

    ```gitignore
    .vscode
    node_modules
    .DS_Store
    ```

1. Create a `src/` folder and a file `index.js` in it

   ```js
   // index.js
   import http from 'http';
   const hostname = '127.0.0.1';
   const port = 3000;

   const server = http.createServer((req, res) => {
     res.writeHead(200, { 'Content-Type': 'text/plain' });
     res.end('Welcome to my REST API!');
   });

   server.listen(port, hostname, () => {
     console.log(`Server running at http://${hostname}:${port}/`);
   });
   ```

1. Write `.editorconfig` and `.prettierrc.js` files according to the examples (see [01-tools-env.md](01-tools-env.md))
1. Test your initial app: `npm run dev`
1. Create a new local repository and setup a remote repository on GitHub and push your current locat project to Github.

    ```sh
    git init
    git add .
    git commit -m "Initial commit"
    git remote add origin <YOUR-REPO-URL>
    git push -u origin main
    ```

`req` variable is an [request object](https://nodejs.org/api/http.html#class-httpclientrequest) containing information about the HTTP request that raised the event. e.g.:

- `req.url`: the request URL string
- `req.method`: the HTTP request method
- `req.headers`: the request headers

In response to `req`, use [response object's](https://nodejs.org/api/http.html#class-httpserverresponse) (`res`) properties/methods to send the desired HTTP response back to the client. e.g.:

- `re.statusCode`: set the response status code
- `res.write()`: add content to response body
- `res.end()`: send te response

Payload data (request body) can be [extracted manually](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction#request-body) from the request stream.

### Simple example of REST API documentation

| Endpoint      | Method | Description                                        | Request Body (Example)            | Response Body (Example)        | Status Codes                         |
|---------------|--------|----------------------------------------------------|----------------------------------|--------------------------------|-------------------------------------|
| `/items`      | GET    | Retrieve a list of all items                       | N/A                              | `[{ "id": 1, "name": "Item1" }, { "id": 2, "name": "Item2" }]` | `200 OK`, `404 Not Found`           |
| `/items`      | POST   | Create a new item                                  | `{ "name": "New Item" }`         | `{ "id": 3, "name": "New Item" }` | `201 Created`, `400 Bad Request`    |
| `/items/:id`  | GET    | Retrieve details of a specific item by its ID      | N/A                              | `{ "id": 1, "name": "Item1" }`  | `200 OK`, `404 Not Found`           |
| `/items/:id`  | PUT    | Update details of a specific item by its ID        | `{ "name": "Updated Item" }`     | `{ "id": 1, "name": "Updated Item" }` | `200 OK`, `400 Bad Request`, `404 Not Found` |
| `/items/:id`  | DELETE | Delete a specific item by its ID                   | N/A                              | N/A                            | `204 No Content`, `404 Not Found`    |

- Endpoint: The URL where the API can be accessed.
- Method: The HTTP verb/method used to interact with the endpoint.
- Description: A brief description of what the endpoint does.
- Request Body: The data sent to the server when making a request (if applicable).
- Response Body: The data returned by the server in response to the request.
- Status Codes: Common HTTP status codes returned by the server. Each status code indicates a different outcome or state of the request.

This is a basic table for documentation. Depending on the complexity of the API, real-world documentation might also include:

- Headers required (e.g., authentication tokens).
- Query parameters for filtering, sorting, or pagination.
- Detailed explanations of each field in the request and response.
- Error message details for different status codes.
- Authentication/Authorization details.

### Using Express.js

<https://expressjs.com/>

>Fast, unopinionated, minimalist web framework for Node.js

Express provides a robust web application feature set and many [other popular frameworks](https://expressjs.com/en/resources/frameworks.html) are built on top of the Express framework.

## Installation

```bash
npm install express
```

Express package is saved as a dependency in the project's `package.json` file.

## Hello world web app

Node.js using built-in http module:

```js
import http from 'http';
const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
   res.writeHead(200, { 'Content-Type': 'text/plain' });
   res.end('Welcome to my REST API!');
});

server.listen(port, hostname, () => {
   console.log(`Server running at http://${hostname}:${port}/`);
});
```

Node with Express:

```js
import express from 'express';
const hostname = '127.0.0.1';
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Welcome to my REST API!');
});

app.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

## Main features

- [Static file server](https://expressjs.com/en/starter/static-files.html)
- [Routing](https://expressjs.com/en/guide/routing.html): define how an application’s endpoints (URIs) respond to client requests
- [Middlewares](https://expressjs.com/en/guide/using-middleware.html): functions that have access to the request object (req), the response object (res) in the application’s request-response cycle.
- Support for several [template engines](https://expressjs.com/en/guide/using-template-engines.html)
  - [Pug](https://pugjs.org/) (formerly Jade) is the default template engine
- [Express application generator](https://expressjs.com/en/starter/generator.html)
  - quick Express app scaffolding, e.g: `npx express-generator --view=pug pug-app`
  - the app structure created by the generator is just one of many ways to structure Express apps and might need a lot of refactoring to suit your needs

### Serving static files

1. Create a folder `src/public` and add any static files into it, e.g. html, css, js, images, etc.
1. Serve the files: `app.use('/static', express.static(path.join(__dirname, 'public')));`
1. Access the files in `public` folder at `http://localhost:3000/static/...`

Note: If using ES modules (`import` statements instead of CommonJS `require()`) and `path.join()` when serving static files, you don't have `__dirname` variable by default. You need set it manually:

```js
import path from 'path';
import {fileURLToPath} from 'url';
...
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

### Serving response in JSON format (for client-side rendering, CSR)

```js
// GET http://localhost:3000/api/resource
app.get('/api/resource', (req, res) => {
  const myData = {title: 'This is an item', description: 'Just some dummy data here'};
  res.json(myData);
});
```

### Reading request parameters

Using route parameters (path variables, the preferred way in REST APIs):

```js
// GET http://localhost:3000/api/resource/99
// property name (id) is set in the route definition
app.get('/api/resource/:id', (req, res) => {
  console.log('path variables', req.params);
  if (req.params.id === '99') {
    const myData = {
      title: 'This is a specific item, id: ' + req.params.id,
      description: 'Just some dummy data here',
    };
    res.json(myData);
  } else {
    res.sendStatus(404);
  }
});
```

Using query parameters:

```js
// GET http://localhost:3000/api/resource?id=99&name=foo
app.get('/api/resource', (req, res) => {
  if (req.query.id === '99') {
   console.log('query params object', req.query);
    const myData = {
      title: 'This is a specific item, id: ' + req.query.id,
      description: 'Just some dummy data here',
    };
    res.json(myData);
  } else {
    res.sendStatus(404);
  }
});
```

### Reading data from request body

```js
// needed for reading request body in JSON format
app.use(express.json());

...
// POST http://localhost:3000/api/resource
// sends request data back to client
app.post('/api/resource', (req, res) => {
  const body = req.body;
  res.status(201);
  res.json({your_request: body});
});
```

---

## Week assignment 1 - Getting started with node.js

Follow the teacher's example (link in Oma) and implement TODOs in the example code. See assignment details in Oma.
