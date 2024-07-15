---
title: "Securely Hashing Passwords Using bcryptjs"
seoTitle: "Password Hashing: bcryptjs Secure Method"
seoDescription: "Securely hash passwords with bcryptjs in Node.js, enhancing storage and verification with hashing and salting for web applications"
datePublished: Wed Jul 10 2024 17:56:27 GMT+0000 (Coordinated Universal Time)
cuid: clyg563su00030ajn45ul3ldh
slug: securely-hashing-passwords-using-bcryptjs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720634090678/348ee8b5-5f5b-47cf-9810-7895256fc581.jpeg
tags: security, nodejs, databases, hashing, bcrypt

---

---

## Introduction

Implementing secure password storage is crucial in web applications, especially when handling sensitive information like credit card numbers or national identification numbers. Storing raw user data as-is in the database is highly insecure because if someone hacks into the database, they can easily access this sensitive information. This is where password hashing comes into play, providing a layer of security by transforming user data into hashed values that are not easily reversible.

## What is Hashing?

Hashing is the process of transforming any given key or string of characters into another value. This value is usually represented by a shorter, fixed-length key that makes it easier to store and retrieve the original string. Unlike encryption, hashing is a one-way process, designed to be irreversible.

## Why Hashing is Needed

Hashing is essential for securely storing passwords in a database. When a user creates an account and sets a password, this password should be stored in a hashed form rather than plain text. This ensures that even if the database is compromised, the actual passwords are not exposed. Hashed passwords cannot be converted back to their original form, making it difficult for attackers to misuse the information.

Additionally, hashing helps in verifying user identities during login without exposing the actual password. When a user attempts to log in, the provided password is hashed and compared with the stored hashed password. If they match, the login is successful, ensuring that the authentication process remains secure and efficient.

### Salting for Enhanced Security

In addition to hashing, salting is a technique used to further enhance the security of hashed passwords. A salt is a randomly generated string added to each password before hashing(often times there are several rounds of hashing), ensuring that even if two users have the same password, their hashed values will differ. bcrypt automatically handles salting internally, making it a robust choice for secure password storage in web applications.

## What is bcrypt?

bcrypt is a popular and secure hashing algorithm designed specifically for password hashing. It incorporates a salt to protect against rainbow table attacks and is computationally intensive to slow down brute-force attacks. bcrypt is widely used in many applications due to its robustness and security features.

## Main Functions of bcrypt

bcrypt provides several key functions essential for password hashing:

### bcrypt.hash()

This function hashes a password.

It takes two arguments: the data to be hashed (e.g., the user's password) and the number of salt rounds.

The number of rounds determines the computational complexity of the hashing process, balancing security and performance.

```javascript
const hashedPassword = await bcrypt.hash(password, 8);
```

### [bcrypt.compare](http://bcrypt.compare)()

This function compares a plain text password with a hashed password.

It takes two arguments: the plain text password and the hashed password.

The function returns a boolean value indicating whether the passwords match.

```javascript
const isMatch = await bcrypt.compare('John2468$', hashedPassword);
```

## Getting Started with bcryptjs

To achieve secure password hashing in Node.js, we'll use the bcryptjs library. bcryptjs is widely known for its strong security and cryptographic reliability.

### Prerequisites

Before proceeding, ensure you have a basic understanding of arrow functions, command line/terminal usage, and async/await.

Also, make sure Node.js is installed on your system.

### Step 1: Install bcryptjs

Initialize your Node.js project, create a JavaScript file (e.g., `index.js`), and install bcryptjs using npm:

```plaintext
npm i bcryptjs
```

### Step 2: Writing the Hashing Logic

Start by requiring bcryptjs in your JavaScript file:

```javascript
const bcrypt = require('bcryptjs');
```

Create an async function to handle the hashing process. This function will hash a user's password and log both the original and hashed passwords:

```javascript
const myFunction = async () => {
    const password = 'John2468$';
    const hashedPassword = await bcrypt.hash(password, 8);
    console.log('Password:', password);
    console.log('Hashed Password:', hashedPassword);
}

myFunction();
```

### Step 3: Comparing Data

bcryptjs allows us to compare a user's input with the stored hashed value using the compare method. This is crucial for verifying passwords during login:

```javascript
const isMatch = await bcrypt.compare('John2468$', hashedPassword);
console.log('Do passwords match?', isMatch);
```

## Practical Implementation in an Express Server

Let's implement bcryptjs in a simple Express server to handle user registration, password hashing, and login verification:

```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

app.use(bodyParser.json());

// In-memory user database
const users = [];

// Registration route
app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 8);
    users.push({ username, password: hashedPassword });
    res.status(201).send('User registered successfully');
});

// Login route
app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    const user = users.find(user => user.username === username);
    
    if (user && await bcrypt.compare(password, user.password)) {
        res.send('Login successful');
    } else {
        res.status(401).send('Invalid username or password');
    }
});

app.listen(port, () => {
    console.log(`Server is running on http://localhost:${port}`);
});
```

## Testing with Postman

### Step 1: Start the Server

Ensure your server is running by executing:

```plaintext
node index.js
```

### Step 2: Register a User

Create a new POST request in Postman with the URL [`http://localhost:3000/register`](http://localhost:3000/register).

Set the request body to JSON and include the username and password:

```json
{
  "username": "john.doe@example.com",
  "password": "John2468$"
}
```

Send the request to register the user.

### Step 3: Login a User

Create a new POST request in Postman with the URL [`http://localhost:3000/login`](http://localhost:3000/login).

Set the request body to JSON and include the same username and password:

```json
{
  "username": "john.doe@example.com",
  "password": "John2468$"
}
```

Send the request to log in. If the credentials are correct, you should receive a "Login successful" message.

## Conclusion

We've demonstrated how to use the bcryptjs library in Node.js to securely hash and compare passwords. By hashing passwords before storing them in the database, we significantly enhance the security of our web applications. This overview covers the basics; for advanced implementations, explore the bcryptjs documentation and experiment with different configurations and use cases, such as a full authentication and authorization workflow.