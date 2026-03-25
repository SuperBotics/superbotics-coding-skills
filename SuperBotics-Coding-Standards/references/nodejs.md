# Node.js Coding Standards — Superbotics

Load `references/javascript.md` first. This file adds Node/Express-specific patterns.

---

## Directory Structure (Node/Express)

```
src/
├── config/             # One config file per concern (database, mail, jwt, app)
├── controllers/        # One file per resource — HTTP layer only
├── services/           # Business logic — one file per domain
├── repositories/       # Database queries — one file per model
├── models/             # ORM model definitions — one file per entity
├── middleware/         # Express middleware — one file per concern
├── routes/             # Route definitions — one file per resource group
├── validators/         # Request validation schemas — one file per resource
├── helpers/            # Pure utility functions — no Express imports allowed
├── constants/          # Shared constants — one file per domain
└── app.js              # Express app setup only — no route logic
server.js               # Server entry point — starts listening only
```

---

## Application Entry Point

- `server.js` only starts the HTTP server — nothing else.
- `app.js` only configures middleware and routes — no business logic.

```javascript
/**
 * File: server.js
 * Purpose: Entry point that starts the HTTP server on the configured port.
 *          Delegates all application setup to app.js.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

'use strict';

const application = require('./src/app');
const applicationConfig = require('./src/config/app-config');

const serverPort = applicationConfig.port;

application.listen(serverPort, function handleServerStart() {
    console.info(`Server started and listening on port ${serverPort}`);
});
```

---

## Controllers

- Receive HTTP request, call a service, return HTTP response.
- Never write database queries or business logic here.
- Always wrap in try/catch and delegate errors to the Express error handler.

```javascript
/**
 * File: user-controller.js
 * Purpose: Handles all HTTP request/response logic for the User resource.
 *          Delegates all data processing to UserService.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

'use strict';

const userService = require('../services/user-service');

/**
 * Handles POST requests to create a new user account.
 *
 * @param {import('express').Request}  request   - Express request object.
 * @param {import('express').Response} response  - Express response object.
 * @param {Function}                   next      - Express next middleware function.
 * @returns {Promise<void>}
 */
async function createUser(request, response, next) {
    try {
        const validatedUserData = request.validatedBody;
        const createdUser = await userService.createNewUser(validatedUserData);

        response.status(201).json({
            message: 'User account created successfully.',
            user: createdUser,
        });
    } catch (serviceError) {
        next(serviceError);
    }
}

module.exports = { createUser };
```

---

## Routes

- One route file per resource group.
- Always apply validation middleware before the controller.
- Name route files to match the resource: `user-routes.js`, `order-routes.js`.

```javascript
/**
 * File: user-routes.js
 * Purpose: Defines all Express routes for the User resource.
 *          Applies authentication and validation middleware before controllers.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

'use strict';

const express = require('express');
const userController = require('../controllers/user-controller');
const validateRequest = require('../middleware/validate-request');
const { createUserSchema } = require('../validators/user-validator');
const authenticateToken = require('../middleware/authenticate-token');

const userRouter = express.Router();

userRouter.post(
    '/',
    authenticateToken,
    validateRequest(createUserSchema),
    userController.createUser
);

module.exports = userRouter;
```

---

## Middleware

- One middleware function per file.
- Middleware must call `next()` or `next(error)` — never both.
- Always name middleware functions (not anonymous) for readable stack traces.

```javascript
/**
 * File: authenticate-token.js
 * Purpose: Validates the JWT Bearer token from the Authorization header.
 *          Attaches the decoded user payload to request.authenticatedUser.
 * Author: Developer Name
 * Created: YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */

'use strict';

const jwt = require('jsonwebtoken');
const jwtConfig = require('../config/jwt-config');

/**
 * Middleware that validates the Authorization Bearer token.
 *
 * @param {import('express').Request}  request   - Express request object.
 * @param {import('express').Response} response  - Express response object.
 * @param {Function}                   next      - Express next middleware function.
 */
function authenticateToken(request, response, next) {
    const authorizationHeader = request.headers['authorization'];

    if (authorizationHeader === undefined) {
        return response.status(401).json({ message: 'Authorization header is missing.' });
    }

    const tokenParts = authorizationHeader.split(' ');
    const bearerToken = tokenParts[1];

    if (bearerToken === undefined) {
        return response.status(401).json({ message: 'Bearer token is missing.' });
    }

    try {
        const decodedPayload = jwt.verify(bearerToken, jwtConfig.secret);
        request.authenticatedUser = decodedPayload;
        next();
    } catch (jwtError) {
        return response.status(403).json({ message: 'Token is invalid or expired.' });
    }
}

module.exports = authenticateToken;
```

---

## Error Handling

- One global error handler middleware registered last in `app.js`.
- All async controller errors must be forwarded via `next(error)`.
- Never let an unhandled promise rejection crash the server — use `process.on('unhandledRejection')`.

---

## Environment and Config

- Never access `process.env` directly in business logic.
- Always proxy environment variables through a dedicated config file.

```javascript
// config/database-config.js
'use strict';

const databaseConfig = {
    host:     process.env.DATABASE_HOST,
    port:     parseInt(process.env.DATABASE_PORT, 10),
    name:     process.env.DATABASE_NAME,
    username: process.env.DATABASE_USER,
    password: process.env.DATABASE_PASSWORD,
};

module.exports = databaseConfig;
```

---

## Forbidden Patterns

- No synchronous file system calls (`fs.readFileSync`) in request handlers.
- No `require()` inside functions — always at the top of the file.
- No hardcoded secrets, tokens, or passwords — always from environment config.
- No `console.log` in production code — use a structured logger (e.g., Winston, Pino).
