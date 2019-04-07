# Netlify Functions Guide

_These instructons were adopted from a [blog article](https://blog.bitsrc.io/react-production-deployment-part-1-netlify-703686631dd1) by Esua Silva_

Netlify provides integration to AWS Lambda functions which enables developers to run server side without a dedicated server. You can learn more about the service in their [official documentation](https://www.netlify.com/docs/functions/).

[Host a Express/Node/React project](#host-a-express/node/react-project)
[Prerequisites](#prerequisites)
[Netlify Command Line Tools](#netlify-command-line-tools)
[Project Setup](#project-setup)
[Backend Setup](#backend-setup)
[Staging Express App](#staging-express-app)
[Setup Netlify on your local machine](#setup-netlify-on-your-local-machine)
[Configure serve and build scripts](#configure-serve-and-build-scripts)
[Frontend Setup](#frontend-setup)

---
## Host a Express/Node/React project

### Prerequisites
- Node v8 or higher installed on your computer
- `create-react-app` installed on your computer
- GitHub account
- Netlify account
- Install the Netlify App on GitHub. Read documentation [here](https://www.netlify.com/docs/github-permissions/).
- Yarn package manager (optional)*

*NOTE: I will use Yarn for my package installation and script commands but you can do the same things using npm.*

### Netlify Command Line Tools

- Install [Netlify's command line interface (CLI)](https://www.netlify.com/docs/cli/) on your workstation
  `npm install netlify-cli -g`

- To login type
  `netlify login`  

  This command will open a browser window asking you to login to Netlify and grant access to Netlify CLI.

### Project Setup
```
├── /api
|  └── server.js
|  └── ...
├── /client
|  └── /node_modules
|  └── package.json
|  └── ...
└── /node_modules
└── package.json
└──....
```

#### Backend Setup
- Install the Node/Express boilerplate (package.json and node_modules) in the root directory
- In addition to installing Express and whatever 3rd party packages you need you will also need to install a package called `serverless-http`
  
  `yarn add serverless-http`

  This module enables your Express.js app to run as a serverless function

- Create a directory named `api` and move your Node server file(s) (server.js or index.js) to that directory.

#### Staging Express App
When staging your Express server to use Netlify functions there are two things to keep in mind.
- Use `express.Router()` class to handle routes and load it as middleware for your app:

  `app.use('/.netlify/functions/server', router);`

- To make Express.js work with Netlify Lambda Functions we need to require the serverless-http package, then use it to wrap the app and export it.

  ```
  module.exports = app;
  module.exports.handler = serverless(app);
  ```

#### Setup Netlify on your local machine
- Create a `netlify.toml` configuration file which tells Netlify how to 
how to build and deploy your site. In the file add the following entries:

```
[build]
  command = "yarn prod"
  functions = "functions"
  publish = "client/build"
```

  You can learn more about additional settings in the `netlify.toml` file [here](https://www.netlify.com/docs/netlify-toml-reference/).

Netlify has a package called `netlify-lambda` that enables you to run your function on your local machine
- Still in the root directory install netlify lambda and serverless-http packages

  `yarn add netlify-lambda --dev`

- You can use serve your functions locally for development by running:
  `netlify-lambda serve <directory>`

  The serve command will start a development server and a watcher on port 9000 (http://localhost:9000).

#### Setup serve and build scripts
- In the root directory open `package.json` and in the `scripts` section replace the default script with following scripts:

  ```
    "start": "yarn start:lambda && yarn start:app",
    "start:app": "cd client && REACT_APP_API_ENDPOINT='http://localhost:9000/' yarn start",
    "start:lambda": "netlify-lambda serve api",
    "prod": "yarn build:lambda; yarn build:app",
    "build:app": "cd client && yarn install && yarn build",
    "build:lambda": "netlify-lambda build api",
  ```

#### Frontend Setup
- In the root directory run `create-react-app client`. You can develop your app as your normally do or copy on existing one.
- To call the API in React we create a global variable for the endpoint URL:
  `const API_ENDPOINT = process.env.REACT_APP_API_ENDPOINT || '/'`*

- Here is an example of how to use it in an api call:
  `axios.get(`${API_ENDPOINT}.netlify/functions/server/api/movies`)...`
  
**NOTE: We are passing a parameter in the when starting the react app that assign the localserver information.*




