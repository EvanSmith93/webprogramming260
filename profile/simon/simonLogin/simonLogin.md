# Simon login

![Simon](../simon.png)

This deliverable demonstrates using MongoDB to persistently store application data, and authenticating users.

You can view this application running here: [Example Simon Login](https://simon-login.cs260.click)

![Simon Login](simonLogin.jpg)

## Database support

This version of Simon calls the database service to save high scores and authorization tokens. This creates a third layer in our Simon technology stack.

1. Client application - Simple HTML/CSS/JavaScript
1. Web service - Caddy, Node.js, Express
1. Database service - MongoDB

### Connecting to the database

We use a cloud service called MongoDB Atlas for our database service. Once we are connected to Atlas, we can make service calls to MongoDB from our web service. This involves specifying the database service endpoint and making database calls like the following.

```Javascript
const { MongoClient } = require('mongodb');

const url = `mongodb+srv://${userName}:${password}@${hostname}`;
const client = new MongoClient(url);
client.connect(err => {
  const collection = client.db("test").collection("devices");

  // ... perform actions on the DB collection

  client.close();
});

```

### Create a MongoDB Atlas cluster

You need to get a MongoDB Atlas account and create a database cluster that you can use as your database service. If you have not done that yet, go back and review the instruction on data services.

### Handling credentials

Make sure you follow the instruction given previously about providing and protecting your MongoDB credentials in a file named `dbConfig.json`. This file will get deployed to production with the `deployService.sh` script.

### Working with the database

The `database.js` file contains the functions for getting and adding high scores. The database functions are called from the `getScores` and `submitScores` endpoints found in `index.js`.

## Authorization

### Authorization UI

The React components found in the `src/login` folder provide all the login UI. Bootstrap provides the styling for the controls.

When the login route is requested from `app.jsx` the `src/login/login.jsx` component is loaded. This determines if the user is already authenticated and loads either the `authenticated.jsx` or `unauthenticated.jsx` component to hide or show the login controls.

When a user logs in, logs out, or creates credentials the service endpoints are called.

### Authorization cookie

The application service uses a secure cookie to store the authorization token for an authenticated user.

```js
function setAuthCookie(res, authToken) {
  res.cookie(authCookieName, authToken, {
    secure: true,
    httpOnly: true,
    sameSite: 'strict',
  });
}
```

Note the use of secure, httpOnly, and sameSite. Make sure you are familiar with what each of these mean.

When a user is logged in, the cookie is added. When a user makes a secure request, the cookie is checked. When the user logs out, the cookie is removed.

### Application service endpoints

The service endpoints are contained in `index.js`. The endpoints include `createAuth`, `getAuth`, and `deleteAuth`. These all work with the database to store and get credentials and update the authorization cookie.

A new Express router, `secureApiRouter` wraps the existing router to add a middleware function that verifies that the authorization cookie is valid before passing the request to endpoints that require authorization. That makes it easy to create secure endpoints by just registering them with `secureApiRouter`.

```js
const secureApiRouter = express.Router();
apiRouter.use(secureApiRouter);

secureApiRouter.use(async (req, res, next) => {
  const authToken = req.cookies[authCookieName];
  const user = await DB.getUserByToken(authToken);
  if (user) {
    next();
  } else {
    res.status(401).send({ msg: 'Unauthorized' });
  }
});

// GetScores
secureApiRouter.get('/scores', async (req, res) => {
  const scores = await DB.getHighScores();
  res.send(scores);
});
```

## Study this code

Get familiar with what this code teaches.

- Clone the repository to your development environment.
  ```sh
  git clone https://github.com/webprogramming260/simon-login.git
  ```
- Set up your Atlas credentials in a file named `dbConfig.json` that is in the same directory as `database.js`.
- Add `dbConfig.json` to your `.gitignore` file so that it doesn't put your credentials into GitHub accidentally.
- Review the code and get comfortable with everything it represents.
- Debug the code in your browser by hosting it from a VS Code debug session. This [video on debugging a node.js based service](https://youtu.be/B0le_Z_2TQY) will step you through the process.

  ⚠ You will no longer use the `live server` extension to launch your frontend code in the browser since your frontend code will now be served up by the Node.js server you created in `index.js`. Set breakpoints in your backend code inside of Visual Studio.

- See how data is populated in the database by viewing the contents of the database using the MongoDB Atlas console.
- Make modifications to the code as desired. Experiment and see what happens.

## Deploy to production

- Deploy to your production environment using the `deployService.sh` script found in the [example class application](https://github.com/webprogramming260/simon-login/blob/main/deployService.sh). Take some time to understand how it works.

  ```sh
  ./deployService.sh -k <yourpemkey> -h <yourdomain> -s simon
  ```

  For example,

  ```sh
  ./deployService.sh -k ~/keys/production.pem -h yourdomain.click -s simon
  ```

- Update your `startup` repository notes.md with what you learned.
- Make sure your project is visible from your production environment (e.g. https://simon.yourdomain.click).
