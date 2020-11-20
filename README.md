## How to create this template from scratch

1. cd to your src folder
    ```
    cd ~/src
    ```

1. generate a new express app without a view
    ```
    npx express-generator --no-view $NEW_PROJECT_NAME
    cd $NEW_PROJECT_NAME
    git init
    ```

1. create .editorconfig with the following contents
    ```
    root = true
    [*]
    indent_style = space
    indent_size = 2
    charset = utf-8
    trim_trailing_whitespace = false
    insert_final_newline = true
    ```

1. create .gitignore with the following contents
    ```
    node_modules/
    yarn-error.log
    .env
    .nyc_output
    coverage
    build/
    yarn-error.log
    ```

1. house cleaning
    ```
    rm routes/users.js
    rm -fr public/
    mv bin/www bin/www.js
    mkdir src
    mv app.js bin/ routes/ src/
    ```

1. add and install modules
    ```
    yarn install
    yarn add dotenv
    yarn add @babel/cli @babel/core @babel/plugin-transform-runtime @babel/preset-env @babel/register @babel/runtime @babel/node nodemon eslint eslint-config-airbnb-base eslint-plugin-import prettier mocha chai nyc sinon-chai supertest coveralls --dev
    ```

1. replace src/routes/index.js with the following
    ```
    import express from 'express';
    import { testEnvironmentVariable } from '../settings';

    const indexRouter = express.Router();

    indexRouter.get('/', (req, res) => res.status(200).json({ message: testEnvironmentVariable }));

    export default indexRouter;
    ```

1. replace src/app.js with the following
    ```
    import logger from 'morgan';
    import express from 'express';
    import cookieParser from 'cookie-parser';
    import indexRouter from './routes/index';

    const app = express();

    app.use(logger('dev'));
    app.use(express.json());
    app.use(express.urlencoded({ extended: true }));
    app.use(cookieParser());
    app.use('/v1', indexRouter);

    export default app; 
    ```

1. replace src/bin/www.js with the following
    ```
    #!/usr/bin/env node

    /**
    * Module dependencies.
    */
    import debug from 'debug';
    import http from 'http';
    import app from '../app';

    /**
    * Normalize a port into a number, string, or false.
    */
    const normalizePort = val => {
      const port = parseInt(val, 10);
      if (Number.isNaN(port)) {
        // named pipe
        return val;
      }
      if (port >= 0) {
        // port number
        return port;
      }
      return false;
    };

    /**
    * Get port from environment and store in Express.
    */
    const port = normalizePort(process.env.PORT || '3000');
    app.set('port', port);

    /**
    * Create HTTP server.
    */
    const server = http.createServer(app);

    /**
    * Event listener for HTTP server "error" event.
    */
    const onError = error => {
      if (error.syscall !== 'listen') {
        throw error;
      }
      const bind = typeof port === 'string' ? `Pipe ${port}` : `Port ${port}`;
      // handle specific listen errors with friendly messages
      switch (error.code) {
        case 'EACCES':
          debug(`${bind} requires elevated privileges`);
          process.exit(1);
          break;
        case 'EADDRINUSE':
          debug(`${bind} is already in use`);
          process.exit(1);
          break;
        default:
          throw error;
      }
    };

    /**
    * Event listener for HTTP server "listening" event.
    */
    const onListening = () => {
      const addr = server.address();
      const bind = typeof addr === 'string' ? `pipe ${addr}` : `port ${addr.port}`;
      debug(`Listening on ${bind}`);
    };

    /**
    * Listen on provided port, on all network interfaces.
    */
    server.listen(port);
    server.on('error', onError);
    server.on('listening', onListening);
    ```

1. create .babelrc
    ```
    {
      "presets": ["@babel/preset-env"],
      "plugins": ["@babel/transform-runtime"]
    }
    ```

1. create nodemon.json file with the following contents
    ```
    {
      "watch": [
        "package.json",
        "nodemon.json",
        ".eslintrc.json",
        ".babelrc",
        ".prettierrc",
        "src/"
      ],
      "verbose": true,
      "ignore": ["*.test.js", "*.spec.js"]
    }
    ```

1. create .eslintrc.json file with the following contents
    ```
    {
      "env": {
        "browser": true,
        "es6": true,
        "node": true,
        "mocha": true
      },
      "extends": ["airbnb-base"],
      "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
      },
      "parserOptions": {
        "ecmaVersion": 2018,
        "sourceType": "module"
      },
      "rules": {
        "indent": ["error", 2, { "SwitchCase": 1 }],
        "linebreak-style": ["error", "unix"],
        "quotes": ["error", "single"],
        "semi": ["error", "always"],
        "no-console": 1,
        "comma-dangle": [0],
        "arrow-parens": [0],
        "object-curly-spacing": ["warn", "always"],
        "array-bracket-spacing": ["warn", "always"],
        "import/prefer-default-export": [0]
      }
    }
    ```

1.  create .prettierrc file with the following contents
    ```
    {
      "trailingComma": "all",
      "tabWidth": 2,
      "semi": true,
      "singleQuote": true,
      "arrowParens": "avoid"
    }
    ```

1. create .env file with the following contents
    ```
    TEST_ENV_VARIABLE="Environment variable is coming across."
    ```

1. create src/settings.js file with the following contents
    ```
    import dotenv from 'dotenv';

    dotenv.config();
    
    export const testEnvironmentVariable = process.env.TEST_ENV_VARIABLE;
    ```

1. create a new test folder
    ```
    mkdir test
    touch test/setup.js
    touch test/index.test.js
    ```

1. populate test/setup.js with the following
    ```
    import supertest from 'supertest';
    import chai from 'chai';
    import sinonChai from 'sinon-chai';
    import app from '../src/app';

    chai.use(sinonChai);
    export const { expect } = chai;
    export const server = supertest.agent(app);
    export const BASE_URL = '/v1';
    ```

1. populate test/index.test.js with the following
    ```
    import { expect, server, BASE_URL } from './setup';

    describe('Index page test', () => {
      it('gets base url', done => {
        server
          .get(`${BASE_URL}/`)
          .expect(200)
          .end((err, res) => {
            expect(res.status).to.equal(200);
            expect(res.body.message).to.equal(
              'Environment variable is coming across.'
            );
            done();
          });
      });
    });
    ```

1. replace scripts section of package.json with the following
    ```
    "scripts": {
      "prestart": "babel ./src --out-dir build",
      "start": "node ./build/bin/www",
      "startdev": "nodemon --exec babel-node ./src/bin/www",
      "lint": "./node_modules/.bin/eslint ./src",
      "pretty": "prettier --write '**/*.{js,json}' '!node_modules/**'",
      "postpretty": "yarn lint --fix",
      "test": "nyc --reporter=html --reporter=text --reporter=lcov mocha -r @babel/register"
    },
    ```
  
1. run pretty and lint the code, ensure there are no errors
    ```
    yarn pretty
    yarn lint
    ```

1. run tests, ensure there are no errors
    ```
    yarn test
    ```

1. start the dev server and open your browser to http://localhost:3000/v1
    ```
    yarn startdev
    ```


