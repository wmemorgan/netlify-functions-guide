# Netlify Functions Guide

_These instructons were adopted from a [Medium article](https://blog.bitsrc.io/react-production-deployment-part-1-netlify-703686631dd1) by Esua Silva_

Netlify provides integration to AWS Lambda functions which enables developers to run server side code without a dedicated server. You can learn more about the service in their [official documentation](https://www.netlify.com/docs/functions/).

## Table of Contents

  - [Deploy Express and React App](#deploy-express-and-react-app)
  - [Prerequisites](#prerequisites)
  - [Netlify Command Line Tools](#netlify-command-line-tools)
  - [Project Setup](#project-setup)
  - [Backend Setup](#backend-setup)
  - [Staging Express App](#staging-express-app)
  - [Setup Netlify on your local machine](#setup-netlify-on-your-local-machine)
  - [Configure serve and build scripts](#configure-serve-and-build-scripts)
  - [Frontend Setup](#frontend-setup)
  - [Deploy to Netlify](#deploy-to-netlify)

---
## Deploy Express and React App

### Prerequisites
- Node v8 or higher installed on your computer
- `create-react-app` installed on your computer
- GitHub account
- Netlify account
- Install the Netlify App on GitHub. Read documentation [here](https://www.netlify.com/docs/github-permissions/).
- Yarn package manager (optional)*

*NOTE: I use yarn for my package installation and script commands but you can do the same things using npm.*

### Netlify Command Line Tools

- Install [Netlify's command line interface (CLI)](https://www.netlify.com/docs/cli/) on your workstation
  `npm install netlify-cli -g`

- To login type
  `netlify login`  

  This command will open a browser window asking you to login to Netlify and grant access to Netlify CLI.

### Project Setup

Here is a layout of the basic file structure:

  ```
  ├── /api
  |  └── server.js
  |  └── ...
  ├── /client
  |  └── /node_modules
  |  └── package.json
  |  └── /public
  |      └── index.html
  |      └── ...
  |  └── /src
  |      └── App.js
  |      └── index.js
  |      └── ...
  └── /node_modules
  └── package.json
  └──....
  ```

#### Backend Setup
- Install the Node/Express boilerplate files (package.json and node_modules) in the root directory
- In addition to installing Express and whatever 3rd party packages you need you will also need to install a package called `serverless-http`
  
  `yarn add serverless-http`

  This module enables your Express.js app to run as a serverless function

- Create a directory named `api` and move your Node server file(s) (server.js or index.js) into that directory.

#### Staging Express App
When staging your Express server to use Netlify functions there are two things to keep in mind.
- Use `express.Router()` class to handle routes and load it as middleware for your app:

  ```javascript
  app.use('/.netlify/functions/server', router);
  ```

- To make Express.js work with Netlify Lambda Functions we need to require the serverless-http package 

  ```javascript
  const serverless = require('serverless-http');
  ```

- Then use it to wrap the app and export it.

  ```javascript
  module.exports = app;
  module.exports.handler = serverless(app);
  ```

#### Setup Netlify on your local machine
- Create a `netlify.toml` configuration file which tells Netlify how to build and deploy your site. In the file add the following entries:

  ```
  [build]
    command = "yarn prod"
    functions = "functions"
    publish = "client/build"

  [[redirects]]
      from = "/*"
      to = "/index.html"
  status = 200  
  ```

**NOTE:** By default Netlify will look for literal pages in a Single Page Application if you refresh your browser page outside of the root directory. The `[[redirects]]` entry is to ensure Netlify properly handles Single Page Applications by redirecting all of the requests to `index.html`. You can learn more about Netlify redirect handling [here](https://www.netlify.com/docs/redirects/).

You can learn more about additional settings in the `netlify.toml` file [here](https://www.netlify.com/docs/netlify-toml-reference/). 

Netlify has a package called `netlify-lambda` that enables you to run your function on your local machine
- While still in the root directory install netlify lambda
  
  `yarn add netlify-lambda --dev`

- You can use serve your functions locally for development by running:
  `netlify-lambda serve <directory>`

The serve command will start a development server and a watcher on port 9000 (http://localhost:9000).

#### Setup serve and build scripts
- In the root directory install a package called `npm-run-all` which will enable you to start the Express server and React app with one command.

  `yarn add npm-run-all --dev`

- Open `package.json` go the `scripts` section and replace the default script with following commands:

  ```javascript
    "start": "run-p start:**",
    "start:app": "cd client && REACT_APP_API_ENDPOINT='http://localhost:9000/' yarn start",
    "start:lambda": "netlify-lambda serve api",
    "prod": "yarn build:lambda; yarn build:app",
    "build:app": "cd client && yarn install && yarn build",
    "build:lambda": "netlify-lambda build api"
  ```

*NOTE: In the `start:app` script we are passing an environment variable `REACT_APP_API_ENDPOINT` which will be used for API calls in our React app.*

#### Frontend Setup
- In the root directory run `create-react-app client`. You can develop your app as your normally do or copy an existing one.
- To call the API in React we create a global variable for the endpoint URL that can be used on our local machine and in a production environment.

  ```javascript
  const API_ENDPOINT = process.env.REACT_APP_API_ENDPOINT || '/'
  ```

- Here is an example of how to use it in an api call:

  ```javascript
  axios.get(`${API_ENDPOINT}.netlify/functions/server/api/movies`)
  ```
  
*NOTE: We are passing an enviroment variable in the start react script that assigns the localserver information.*

#### Deploy to Netlify
Setup continuous deployment where push commits to your GitHub repository will trigger the app to build and deploy. 

- Connect your repository to Netlify by running the following command from your local repository:

  `netlify init`

You’ll be prompted to log in to your GitHub account, which will create an account-level access token in your home directory under `.netlify/config.json`. Detailed information regarding continuous deployment can be found in the [documentation](https://www.netlify.com/docs/cli/#continuous-deployment).



