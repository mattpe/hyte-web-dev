# Express Server Integration to Kubios cloud

The idea is to combine the Kubios cloud API data with the local database data. The Kubios cloud API is used for authentication and data analysis. The local database is used for storing user data and other data that is not stored in the Kubios cloud.

Technically, the client sends requests to the backend server. The backend server acts as a proxy and forwards the requests to the Kubios cloud API. The backend server also stores the client id (api key) and other sensitive information. The client does not need to know the Kubios cloud API details. CORS issues are also avoided.

## Links

- [Kubios Cloud](https://www.kubios.com/kubios-cloud/)
- [Kubios Cloud API documentation](https://analysis.kubioscloud.com/v2/portal/documentation/index.html)

## Implement Login using Kubios cloud

1. Continue the node express project from the previous course
1. Add Kubios settings to `.env` file (and to `.env.sample`)

    ```conf
    ...
    JWT_EXPIRES_IN=1h # 1 hour is the default for Kubios too?
    KUBIOS_CLIENT_ID=get-from-Oma-docs # DO NOT COMMIT THIS TO GIT or share publicly
    KUBIOS_LOGIN_URL=https://kubioscloud.auth.eu-west-1.amazoncognito.com/login
    KUBIOS_REDIRECT_URI=https://analysis.kubioscloud.com/v1/portal/login
    KUBIOS_API_URI=https://analysis.kubioscloud.com/v2
    KUBIOS_USER_AGENT=YOUR_APP_NAME_HERE
    ```

1. Add a function to `userModel.mjs` to check if user email already exists in local database

    ```js
    const selectUserByEmail = async (email) => {
      try {
        const sql = 'SELECT * FROM Users WHERE email=?';
        const params = [email];
        const [rows] = await promisePool.query(sql, params);
        // console.log(rows);
        // if nothing is found with the user id, result array is empty []
        if (rows.length === 0) {
          return {error: 404, message: 'user not found'};
        }
        // Remove password property from result
        delete rows[0].password;
        return rows[0];
      } catch (error) {
        console.error('selectUserByEmail', error);
        return {error: 500, message: 'db error'};
      }
    };
    ```

1. Create a new auth controller to use the Kubios cloud user credentials for authentication `kubios-auth-controller.mjs`:

    ```js
    /**
     * Authentication resource controller using Kubios API for login
    * @module controllers/auth-controller
    * @author mattpe <mattpe@metropolia.fi>
    * @requires jsonwebtoken
    * @requires bcryptjs
    * @requires dotenv
    * @requires models/user-model
    * @requires middlewares/error-handler
    * @exports postLogin
    * @exports getMe
    */

    import 'dotenv/config';
    import jwt from 'jsonwebtoken';
    import fetch from 'node-fetch';
    import {v4} from 'uuid';
    import {customError} from '../middlewares/error-handler.mjs';
    import {
      insertUser,
      selectUserByEmail,
      selectUserById,
    } from '../models/user-model.mjs';

    // Kubios API base URL should be set in .env
    const baseUrl = process.env.KUBIOS_API_URI;

    /**
    * Creates a POST login request to Kubios API
    * @async
    * @param {string} username Username in Kubios
    * @param {string} password Password in Kubios
    * @return {string} idToken Kubios id token
    */
    const kubiosLogin = async (username, password) => {
      const csrf = v4();
      const headers = new Headers();
      headers.append('Cookie', `XSRF-TOKEN=${csrf}`);
      headers.append('User-Agent', process.env.KUBIOS_USER_AGENT);
      const searchParams = new URLSearchParams();
      searchParams.set('username', username);
      searchParams.set('password', password);
      searchParams.set('client_id', process.env.KUBIOS_CLIENT_ID);
      searchParams.set('redirect_uri', process.env.KUBIOS_REDIRECT_URI);
      searchParams.set('response_type', 'token');
      searchParams.set('scope', 'openid');
      searchParams.set('_csrf', csrf);

      const options = {
        method: 'POST',
        headers: headers,
        redirect: 'manual',
        body: searchParams,
      };
      let response;
      try {
        response = await fetch(process.env.KUBIOS_LOGIN_URL, options);
      } catch (err) {
        console.error('Kubios login error', err);
        throw customError('Login with Kubios failed', 500);
      }
      const location = response.headers.raw().location[0];
      // console.log(location);
      // If login fails, location contains 'login?null'
      if (location.includes('login?null')) {
        throw customError(
          'Login with Kubios failed due bad username/password',
          401,
        );
      }
      // If login success, Kubios response location header
      // contains id_token, access_token and expires_in
      const regex = /id_token=(.*)&access_token=(.*)&expires_in=(.*)/;
      const match = location.match(regex);
      const idToken = match[1];
      return idToken;
    };

    /**
    * Get user info from Kubios API
    * @async
    * @param {string} idToken Kubios id token
    * @return {object} user User info
    */
    const kubiosUserInfo = async (idToken) => {
      const headers = new Headers();
      headers.append('User-Agent', process.env.KUBIOS_USER_AGENT);
      headers.append('Authorization', idToken);
      const response = await fetch(baseUrl + '/user/self', {
        method: 'GET',
        headers: headers,
      });
      const responseJson = await response.json();
      if (responseJson.status === 'ok') {
        return responseJson.user;
      } else {
        throw customError('Kubios user info failed', 500);
      }
    };

    /**
    * Sync Kubios user info with local db
    * @async
    * @param {object} kubiosUser User info from Kubios API
    * @return {number} userId User id in local db
    */
    const syncWithLocalUser = async (kubiosUser) => {
      // Check if user exists in local db
      let userId;
      const result = await selectUserByEmail(kubiosUser.email);
      // If user with the email not found, create new user, otherwise use existing
      if (result.error) {
        // Create user
        const newUser = {
          username: kubiosUser.email,
          email: kubiosUser.email,
          // Random password, quick workaround for the required field
          password: v4(),
        };
        const newUserResult = await insertUser(newUser);
        userId = newUserResult.user_id;
      } else {
        userId = result.user_id;
      }
      console.log('syncWithLocalUser userId', userId);
      return userId;
    };

    /**
    * User login
    * @async
    * @param {object} req
    * @param {object} res
    * @param {function} next
    * @return {object} user if username & password match
    */
    const postLogin = async (req, res, next) => {
      const {username, password} = req.body;
      // console.log('login', req.body);
      try {
        // Try to login with Kubios
        const kubiosIdToken = await kubiosLogin(username, password);
        const kubiosUser = await kubiosUserInfo(kubiosIdToken);
        const localUserId = await syncWithLocalUser(kubiosUser);
        // Include kubiosIdToken in the auth token used in this app
        // NOTE: What is the expiration time of the Kubios token?
        const token = jwt.sign(
          {userId: localUserId, kubiosIdToken},
          process.env.JWT_SECRET,
          {
            expiresIn: process.env.JWT_EXPIRES_IN,
          },
        );
        return res.json({
          message: 'Logged in successfully with Kubios',
          user: kubiosUser,
          user_id: localUserId,
          token,
        });
      } catch (err) {
        console.error('Kubios login error', err);
        return next(err);
      }
    };

    /**
    * Get user info based on token
    * @async
    * @param {object} req
    * @param {object} res
    * @return {object} user info
    */
    const getMe = async (req, res) => {
      const user = await selectUserById(req.user.userId);
      res.json({user, kubios_token: req.user.kubiosIdToken});
    };

    export {postLogin, getMe};
    ```

1. Replace existing auth controller with new one in `authRouter.mjs`:

    ```js
    // import {getMe, postLogin} from '../controllers/auth-controller.mjs';
    import {getMe, postLogin} from '../controllers/kubios-auth-controller.mjs';
    ```

1. Test login with Postman using your Kubios cloud user account

## Example of requesting data from Kubios cloud

1. Add a new controller for Kubios cloud data `kubios-controller.mjs`:

    ```js
    import 'dotenv/config';
    import fetch from 'node-fetch';
    // import {customError} from '../middlewares/error-handler.mjs';

    // Kubios API base URL should be set in .env
    const baseUrl = process.env.KUBIOS_API_URI;

    /**
    * Get user data from Kubios API example
    * TODO: Implement error handling
    * @async
    * @param {Request} req Request object including Kubios id token
    * @param {Response} res
    * @param {NextFunction} next
    */
    const getUserData = async (req, res, next) => {
      const {kubiosIdToken} = req.user;
      const headers = new Headers();
      headers.append('User-Agent', process.env.KUBIOS_USER_AGENT);
      headers.append('Authorization', kubiosIdToken);

      const response = await fetch(
        // TODO: set the from date in request parameters
        baseUrl + '/result/self?from=2022-01-01T00%3A00%3A00%2B00%3A00',
        {
          method: 'GET',
          headers: headers,
        },
      );
      const results = await response.json();
      return res.json(results);
    };

    /**
    * Get user info from Kubios API example
    * TODO: Implement error handling
    * @async
    * @param {Request} req Request object including Kubios id token
    * @param {Response} res
    * @param {NextFunction} next
    */
    const getUserInfo = async (req, res, next) => {
      const {kubiosIdToken} = req.user;
      const headers = new Headers();
      headers.append('User-Agent', process.env.KUBIOS_USER_AGENT);
      headers.append('Authorization', kubiosIdToken);

      const response = await fetch(baseUrl + '/user/self', {
        method: 'GET',
        headers: headers,
      });
      const userInfo = await response.json();
      return res.json(userInfo);
    };

    export {getUserData, getUserInfo};
    ```

1. Add router for Kubios controller `kubios-router.mjs`:

    ```js
    import express from 'express';
    import {authenticateToken} from '../middlewares/authentication.mjs';
    import {getUserData, getUserInfo} from '../controllers/kubios-controller.mjs';

    const kubiosRouter = express.Router();

    kubiosRouter
      .get('/user-data', authenticateToken, getUserData)
      .get('/user-info', authenticateToken, getUserInfo);

    export default kubiosRouter;
    ```

1. Add the router to `index.js`: 

    ```js
    ...
    // Kubios API resource (/api/kubios)
    app.use('/api/kubios', kubiosRouter);
    ...
    ```

1. Test routes with Postman using token got from Kubios cloud login

## Considerations & enhancements

- Fix/refactor all routes/controllers that use other info besides the `userId` and `kubiosIdToken` extracted from JWT authentication token
- Add error handling to Kubios controller
- What is the best way to store the user data?
  - Note that kubios data is not stored in the database in these examples
  - How to store or integrate additional data in local database?
- Update the database?
  - Remove passwords from the database if not needed anymore
