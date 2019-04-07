# Netlify Functions Guide
Netlify provides integration to AWS Lambda functions which enables developers to run server side without a dedicated server. You can learn more about the service in their [official documentation](https://www.netlify.com/docs/functions/).

## Host a React/Node/Express project

### Prerequisites
- Node v8 or higher installed on your computer
- `create-react-app` installed on your computer
- GitHub account
- Netlify account
- Install the Netlify App on GitHub. Read documentation [here](https://www.netlify.com/docs/github-permissions/).
- Yarn package manager (optional)*

*I will use Yarn for my package installation and script commands but you can do the same things using npm*

### Netlify Command Line Tools (macOS)

- Install [Netlify's command line interface (CLI)](https://www.netlify.com/docs/cli/) on your workstation
  `npm install netlify-cli -g`

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

#### Backend
- Have the Node/Express boilerplate (package.json and node_modules) in the root directory
- Place the Node entry point file (server.js or index.js) in subdirectory (name is "api").

#### Frontend
- In the root directory run `create-react-app client`.


