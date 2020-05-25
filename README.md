# Overview

Express is a powerful but flexible Javascript framework for creating web servers and APIs. It can be used for everything from simple static file servers to JSON APIs to full production servers.

## Starting a Server

To create a server, the imported express function must be invoked.
```js
const express = require('express');
const app = express();
```
- On the first line, we import the Express library with `require`. 
- When invoked on the second line, it returns an instance of an Express application. 
- This application can then be used to start a server and specify server behavior.

The purpose of a server is to listen for requests, perform whatever action is required to satisfy the request, and then return a response.

In order for our server to start responding, we have to tell the server where to *listen* for new requests by providing a port number argument to a method called `app.listen()`.

The server will then listen on the specified port and respond to any requests that come into it.

The second argument is a callback function that will be called once the server is running and ready to receive responses:
```js
const PORT = 4001;
app.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});
```

## Routes

### Writing a Route

To tell our server how to deal with any given request, we register a series of *routes*. Routes define the control flow for requests based on the request’s path and HTTP verb.

Express uses `app.get()` to register routes to match GET requests. 

Express routes (including `app.get()`) usually take two arguments:
- a path (usually a string), and 
- a callback function to handle the request and send a response.
```js
const moods = [{ mood: 'excited about express!'}, { mood: 'route-tastic!' }];
app.get('/moods', (req, res, next) => {
  // Here we would send back the moods array in response
});
```
The route above will match any GET request to `'/moods'` and call the callback function, passing in two objects as the first two arguments.

### Sending a Response

HTTP follows a one request-one response cycle. Each client expects exactly one response per request, and each server should only send a single response back to the client per request.

Express servers send responses using the `.send()` method on the response object. `.send()` will take any input and include it in the response body.
```js
const monsters = [{ type: 'werewolf' }, { type: 'hydra' }, { type: 'chupacabra' }];
app.get('/monsters', (req, res, next) => {
  res.send(monsters);
});
```
- a GET `/monsters` request will match the route, Express will call the callback function, and 
- the `res.send()` method will send back an array of spooky monsters.

In addition to `.send()`, `.json()` can be used to explicitly send JSON-formatted responses.

### Matching Route Paths

Express tries to match requests by route, meaning that if we send a request to `<server address>:<port number>/api-endpoint`, the Express server will search through any registered routes in order and try to match `/api-endpoint`.

Express searches through routes in the order that they are registered in your code. The first one that is matched will be used, and its callback will be called.

### Route Parameters

Parameters are route path segments that begin with `:` in their Express route definitions. They act as wildcards, matching any text at that path segment. For example `/monsters/:id` will match both `/monsters/1` and `/monsters/45`.

Express parses any parameters, extracts their actual values, and attaches them as an object to the request object: `req.params`. This object’s keys are any parameter names in the route, and each key’s value is the actual value of that field per request.

```js
const monsters = { hydra: { height: 3, age: 4 }, dragon: { height: 200, age: 350 } };
// GET /monsters/hydra
app.get('/monsters/:name', (req, res, next) => {
  console.log(req.params) // { name: 'hydra' };
  res.send(monsters[req.params.name]);
});
```
- A `.get()` route is defined to match `/monsters/:name` path.
- When a GET request arrives for `/monsters/hydra`, the callback is called.
- Inside the callback, `req.params` is an object with the key name and the value hydra, which was present in the actual request path.
- The appropriate monster is retrieved by its name from the monsters object and sent back to the client.

### Setting Status Codes

Response codes provide information to clients about how their requests were handled. For example, any `res.send()` has by default sent a 200 OK status code.

The res object has a `.status()` method to allow us to set the status code, and other methods like `.send()` can be chained from it.
```js
const monsterStoreInventory = { fenrirs: 4, banshees: 1, jerseyDevils: 4, krakens: 3 };
app.get('/monsters-inventory/:name', (req, res, next) => {
  const monsterInventory = monsterStoreInventory[req.params.name];
  if (monsterInventory) {
    res.send(monsterInventory);
  } else {
    res.status(404).send('Monster not found');
  }
});
```
- Here, we've implemented a route to retrieve inventory levels from a Monster Store.
- Inventory levels are kept in the `monsterStoreInventory` variable. 
- When a request arrives for `/monsters-inventory/mothMen`, the route matches and so the callback is invoked.
- `req.params.name` will be equal to 'mothMen' and so our program accesses `monsterStoreInventory['mothMen']`.
- Since there are no `mothMen` in our inventory, `res.status()` sets a 404 status code on the response, and `.send()` sends the response.

### Other HTTP Methods

`PUT` requests are used for updating existing resources.

#### Using Queries

`Query strings` appear at the end of the path in URLs, and they are indicated with a `?` character. 
For instance, in `/monsters/1?name=chimera&age=1`, the query string is `name=chimera&age=1` and the path is `/monsters/1/`.

Query strings do not count as part of the route path. Instead, the Express server parses them into a JavaScript object and attaches it to the request body as the value of `req.query`.

The key: value relationship is indicated by the `=` character in a query string, and key-value pairs are separated by `&`.

In the above example route, the `req.query` object would be `{ name: 'chimera', age: '1' }`.

```js
const monsters = { '1': { name: 'cerberus', age: '4'  } };
// PUT /monsters/1?name=chimera&age=1
app.put('/monsters/:id', (req, res, next) => {
  const monsterUpdates = req.query;
  monsters[req.params.id] = monsterUpdates;
  res.send(monsters[req.params.id]);
});
```
- Here, we have a route for updating monsters by ID.
- When a PUT `/monsters/1?name=chimera&age=1` request arrives, our callback function is called and, we create a `monsterUpdates` variable to store `req.query`.
- Since `req.params.id` is `'1'`, we replace `monsters['1']`‘s value with `monsterUpdates`.
- Finally, Express sends back the new `monsters['1']`.

When updating, many servers will send back the updated resource after the updates are applied so that the client has the exact same version of the resource as the server and database.


`POST` is the HTTP method verb used for creating new resources. Because `POST` routes create new data, their paths do not end with a route parameter, but instead end with the type of resource to be created.

Express uses `.post()` as its method for POST requests. 

`DELETE` is the HTTP method verb used to delete resources. Because `DELETE` routes delete currently existing data, their paths should usually end with a route parameter to indicate which resource to delete.

Express uses `.delete()` as its method for DELETE requests. Servers often send a 204 No Content status code if deletion occurs without error.

### Express Routers

Routers are mini versions of Express applications — they provide functionality for handling route matching, requests, and sending responses, but they do not start a separate server or listen on their own ports.

Routers use all the `.get()`, `.put()`, `.post()`, and `.delete()` routes that you know and love.

An Express router provides a subset of Express methods. To create an instance of one, we invoke the `.Router()` method on the top-level Express import.

To use a router, we *mount* it at a certain path using `app.use()` and pass in the router as the second argument. This router will now be used for all paths that begin with that path segment. 

To create a router to handle all requests beginning with `/monsters`, the code would look like this:
```js
const express = require('express');
const app = express();

const monsters = {
  '1': {
    name: 'godzilla',
    age: 250000000
  },
  '2': {
    Name: 'manticore',
    age: 21
  }
}

const monstersRouter = express.Router();

app.use('/monsters', monstersRouter);

monstersRouter.get('/:id', (req, res, next) => {
  const monster = monsters[req.params.id];
  If (monster) {
    res.send(monster);
  } else {
    res.status(404).send();
  }
});
```
- Inside the `monstersRouter`, all matching routes are assumed to have `/monsters` prepended, as it is mounted at that path.
- `monstersRouter.get('/:id')` matches the full path `/monsters/:id`.
- When a `GET` `/monsters/1` request arrives, Express matches `/monsters` in `app.use()` because the beginning of the path (`'/monsters'`) matches.
- Express’ route-matching algorithm enters the `monstersRouter`‘s routes to search for full path matches. 
- Since `monstersRouter.get('/:id')` is mounted at `/monsters`, the two paths together match the entire request path (`/monsters/1`), so the route matches and the callback is invoked. 
- The `'godzilla'` monster is fetched from the monsters array and sent back.

### Middleware

Middleware is code that executes between a server receiving a request and sending a response. It operates on the boundary, so to speak, between those two HTTP actions.

In Express, middleware is a function. Middleware can perform logic on the request and response objects, such as: 
- inspecting a request, 
- performing some logic based on the request, 
- attaching information to the response, 
- attaching a status to the response, 
- sending the response back to the user, or 
- simply passing the request and response to another middleware.

```js
app.use((req, res, next) => {
  console.log('Request received');
});
```
- `app.use()` takes a callback function that it will call for every received request.
- Every time the server receives a request, it will find the first registered middleware function and call it.
- The server will find the callback function specified above, call it, and print out 'Request received'.

To quote the Express documentation:
`An Express application is essentially a series of middleware function calls.`

#### next()

The middleware stack is processed in the order they appear in the application file, such that middleware defined later happens after middleware defined before. It’s important to note that this is regardless of method — an `app.use()` that occurs after an `app.get()` will get called after the `app.get()`.

An Express middleware is a function with three parameters: `(req, res, next)`. The sequence is expressed by a set of callback functions invoked progressively after each middleware performs its purpose. 

The third argument to a middleware function, `next`, should get explicitly called as the last part of the middleware’s body. This will hand off the processing of the request and the construction of the response to the next middleware in the stack.