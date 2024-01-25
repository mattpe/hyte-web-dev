# Node.js & Express development continues

- [Getting started with Node.js](https://nodejs.dev/en/learn/) - Node.js official documentation
- [Express.js](https://expressjs.com/) - Express.js official documentation

## Creating a REST API with mock data

1. Continue the Node project started last week (`git checkout week1`). Create a new git branch `week2` and choose it (`git checkout -b week2`).
1. Make sure you have all packages (listed as dependencies in `package.json`) installed (`npm install`).
1. Create a new module file `src/items.mjs`
   - move the `items` mock data array from `src/index.js` to the new module file
   - create a new function `getItems()` in the module and export it

      ```js
      const getItems = (req, res) => {
        res.json(items);
      }
      export {getItems};
      ```

   - import the `getItems()` function in `src/index.js` and use it to get the items data by modifying the `GET /items` endpoint

      ```js
      import {getItems} from './items.mjs';
      ...
      app.get('/items', getItems);
      ```

   - do the same for the `GET /items/:id` and `POST /items` endpoints (create functions in `src/items.mjs` and import and use them in `src/index.js`)

1. Implement a functional [simple example REST API endpoints](02-node-express.md#simple-example-of-rest-api-documentation) for the item mock data in `src/items.js`. For example:
   - Adding an item should push a new item object to the mock data array
     - See the [example](02-node-express.md#reading-data-from-request-body) of reading POST body data in JSON format 
   - Deleting an item should delete the item object with matching id
   - Invalid http requests should be responded with appropriate messages and status codes
1. Create a new module file `src/users.mjs` and add the following mock data to it:

    ```js
    const users = [
      {
        id: 1,
        username: "johndoe",
        password: "password1",
        email: "johndoe@example.com"
      },
      {
        id: 2,
        username: "janedoe",
        password: "password2",
        email: "janedoe@example.com"
      },
      {
        id: 3,
        username: "bobsmith",
        password: "password3",
        email: "bobsmith@example.com"
      }
    ];
    ```

1. Design and implement API endpoints for `users` resource using the mock data provided
   - figure out and add correct status codes and corresponding messages or data to the responses
   - use `POST` method to create a new user
   - use `GET` method to get a specific user
   - create a dummy login endpoint using `POST` method
     - check that the username and password match and return a success message if they do or an error message if they don't
1. Import and use the `users` module in `src/index.js` to register the new endpoints

---

## Week assignment 2

Follow the teacher's example ("Creating a REST API with mock data" section above, link to the example code is in Oma) and implement TODOs in the example code. See assignment details in Oma.
