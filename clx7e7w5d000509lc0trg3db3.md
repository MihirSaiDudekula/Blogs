---
title: "How to Implement a Simple Rate Limiter in Express using Middlewares"
datePublished: Sun Jun 09 2024 10:20:09 GMT+0000 (Coordinated Universal Time)
cuid: clx7e7w5d000509lc0trg3db3
slug: how-to-implement-a-simple-rate-limiter-in-express-using-middlewares
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/yLDabpoCL3s/upload/faa2e2c1fd4229450763ecdbed3c03b3.jpeg
tags: express, javascript, servers, middleware

---

Recently, while working with Express middlewares, I came across an important concept: a middleware placed before all requests runs every time a request is made to any route. While I was wondering what could be an interesting application of this concept, I stumbled upon the idea of making a simple a rate limiter middleware.

Rate limiting is a crucial part of modern web application security and performance management. It helps in preventing abuse by limiting the number of requests a user can make to a server within a given time frame. In this article, we'll discuss how to create a simple rate limiter in an Express.js application using middleware.

The task in hand is pretty simple:

We want to create a global middleware (app.use) which will rate limit the requests from a user to only 5 request per second If a user sends more than 5 requests in a single second, the server should block them with a 404. User will be sending in their user id in the header as 'user-id', which uniquely identifies each user. We have a numberOfRequestsPerUser object to start off with, which clears every one second.

## Step-by-Step Guide

### 1\. Setting Up Express

First, we need to set up an Express server. We’ll also need some basic routes for demonstration.

```javascript
const express = require('express');
const app = express();

app.get('/user', (req, res) => {
  res.status(200).json({ name: 'john' });
});

app.post('/user', (req, res) => {
  res.status(200).json({ message: 'created sample user' });
});

module.exports = app;
```

### 2\. Creating the Rate Limiter Middleware

We’ll create a middleware to limit each user to 5 requests per second. If a user exceeds this limit, they will receive a 404 response.

```javascript
const numberOfRequestsPerUser = {};

app.use((req, res, next) => {
  const userId = req.headers['user-id'];
  if (!userId) {
    return res.status(400).send('Missing user-id header');
  }
  if (numberOfRequestsPerUser[userId]) {
    numberOfRequestsPerUser[userId]++;
    if (numberOfRequestsPerUser[userId] > 5) 
    {
      return res.status(404).send("rate limit exceeded");
    }
  } else {
    numberOfRequestsPerUser[userId] = 1;
  }
  next();
});

setInterval(() => {
  numberOfRequestsPerUser = {};
}, 1000);
```

### 3\. How the Rate Limiter Works

* **Initial State**: When a request comes in, we check the `user-id` in the request headers.
    
* **First Request**: If the `user-id` does not exist in our tracking object, we initialize it with a count of 1.
    
* **Subsequent Requests**: For each subsequent request within the same second, we increment the count for that `user-id`.
    
* **Exceeding the Limit**: If the count exceeds 5, we respond with a 404 status and a "rate limit exceeded" message.
    
* **Resetting Counts**: Every second, we reset the request counts to allow new requests.
    

### Example Walkthrough

Let's see how this works with an example user sending requests:

1. **First Request**
    
    * Headers: `{ "user-id": "123" }`
        
    * State: `{ "123": 1 }`
        
2. **Second Request**
    
    * Headers: `{ "user-id": "123" }`
        
    * State: `{ "123": 2 }`
        
3. **Third, Fourth, and Fifth Requests**
    
    * Headers: `{ "user-id": "123" }`
        
    * The state is incremented for each request: `{ "123": 3 }`, `{ "123": 4 }`, `{ "123": 5 }`
        
4. **Sixth Request**
    
    * Headers: `{ "user-id": "123" }`
        
    * State: `{ "123": 6 }`
        
    * Response: `404 {"message": "rate limit exceeded"}`.
        
5. **New Second Interval**
    
    * State is reset.
        
    * The user can make 5 new requests in the new second.
        

### Why Use an Object for Tracking Requests?

Using an object allows us to track the number of requests from multiple users at the same time. For instance, if the tracking object is `{ "123": 4, "456": 2 }`, it means user "123" has made 4 requests, and user "456" has made 2 requests. This structure ensures efficient tracking and limiting of requests per user.

### Conclusion

Middlewares let us intercept and process requests before they reach the main application logic. This flexibility helps us enforce rules, handle authentication, and manage different parts of request handling smoothly. Additionally, middleware helps us keep our code organized and easy to maintain, allowing us to add or change features easily. Middleware is essential for building strong, scalable, and secure applications, making it a key tool in building Express servers.