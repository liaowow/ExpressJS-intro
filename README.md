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
