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

