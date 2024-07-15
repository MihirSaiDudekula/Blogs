---
title: "How JWT Authentication Works: Explained Simply"
datePublished: Thu Jun 13 2024 13:49:56 GMT+0000 (Coordinated Universal Time)
cuid: clxdbh32a000109l35zcqaiq7
slug: how-jwt-authentication-works-explained-simply
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1718124891587/4d6045e1-6e36-40e7-b220-03553cf739ba.png
tags: authentication, jwt, jwtauthentication

---

Implementing authentication in web applications can be a challenging task. Continuously making calls to the database for every request is not only impractical but can also lead to performance bottlenecks. The complexity increases further when you need to provide authenticated users with selective access to certain content that should remain inaccessible otherwise.

This is where JSON Web Tokens (JWT) come into play. JWT is a standard that defines a compact and self-contained way of securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed, either with a secret or a public/private key pair. JWTs streamline the authentication process, reducing the need for constant database queries and enabling fine-grained access control.

### How JWT Works

1. **Authentication**: We first want the user to log in using their credentials. It is at this point, given that the credentials are correct, we want the server to create a JWT, which looks something like this
    

```plaintext
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.SflKxwRJSMeKKF2QT4fwpMeJf
```

A JWT contains three parts:

1. **Header**:
    
    ```plaintext
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
    ```
    
    * Contains information about the type of the token, which is JWT, and the signing algorithm being used, such as HMAC SHA256 or RSA.
        
2. **Payload**:
    
    ```plaintext
    eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
    ```
    
    * Contains the payload, or the actual information that is encoded. This typically includes claims about the user or any additional data needed by the application.
        
3. **Signature**:
    
    ```plaintext
    SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    ```
    
    * Created by hashing the header and payload together with a secret key using the specified algorithm. The signature is used to verify the authenticity and integrity of the token.
        
4. **Storing**: The client stores the JWT, usually in local storage or cookies.
    
5. **Authorization**: Every subsequent request from the client to the server includes the JWT, usually in the HTTP header.
    
6. **Verification**: The server verifies the JWT using the secret key or the public key. If the token is valid, the server processes the request.
    

### Advantages of JWT

1. **Compact**: Because of their size, JWTs can be sent through a URL, POST parameter, or inside an HTTP header, which makes them very versatile.
    
2. **Self-contained**: The payload contains all the required information about the user, eliminating the need to query the database multiple times.
    
3. **Secure**: Since JWTs can be signed using public/private key pairs, the identity of the sender can be verified. Additionally, the integrity of the claims can be checked to ensure they haven’t been tampered with.
    

### Use Cases

* **Authorization**: Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token.
    
* **Information Exchange**: JWTs are a good way of securely transmitting information between parties. Since they can be signed, you can be sure the senders are who they say they are.
    

JWTs are base64 URL-encoded strings that anyone can decode. The payload contains the claims, such as user information, expiration time, etc. However, the crucial part of a JWT is its signature. While anyone can decode the token and read its contents, only those with the secret key (or public key, in the case of asymmetric encryption) can verify that the token is valid and has not been tampered with.

### Using JWT in our Express server

The `jsonwebtoken` library in Node.js is used to create and verify JSON Web Tokens (JWTs). It contains multiple useful functions, but the few that are of important to us include token signing, token verification and token expiry.

### Key Features of this library

* **Signing Tokens**: You can create a token with a given payload and a secret key.
    
* **Verifying Tokens**: You can verify the token to ensure its integrity and authenticity using the existing secret key.
    
* **Expiration**: You can set an expiration time for the token to enhance security.
    

### Installation

We can install the library via npm:

```bash
npm install jsonwebtoken
```

### Syntax for Generating (Signing) Tokens

To generate a JWT, you use the `sign` method. This method takes a payload, a secret key, and an optional configuration object.

#### Example Code for Generating a JWT

```javascript
const jwt = require('jsonwebtoken');

// Secret key for signing the token
const secretKey = 'your-secret-key-here';

// Payload to be included in the token
const payload = {
  username: 'johndoe',
  role: 'user'
};

// Options for the token
const options = {
  expiresIn: '1h' // Token expiration time
};

// Generate the token
const token = jwt.sign(payload, secretKey, options);
/*the sign function takes 3 arguments,(payload, secretKey and options) 
and creates and returns the token*/

console.log('Generated Token:', token);
```

#### Parameters for `jwt.sign`

1. **payload**: An object containing the data you want to include in the token.
    
2. **secretKey**: A string used to sign the token.
    
3. **options**: (Optional) An object specifying additional settings, such as `expiresIn`.
    

### Syntax for Verifying Tokens

To verify a JWT, you use the `verify` method. This method takes the token, the secret key, and an optional callback function.

#### Example Code for Verifying a JWT

```javascript
const jwt = require('jsonwebtoken');

// The token to be verified
const token = 'your-jwt-token';

// Secret key used for signing the token
const secretKey = 'your-secret-key';

try {
  // Verify the token
  const decoded = jwt.verify(token, secretKey);

  // If successful, decoded will contain the payload
  console.log('Decoded Payload:', decoded);
} catch (err) {
  // If the token is invalid or expired, an error will be thrown
  console.error('Token verification failed:', err.message);
}
```

#### Parameters for `jwt.verify`

1. **token**: The JWT to be verified.
    
2. **secretKey**: The secret key that was used to sign the token.
    
3. **callback**: (Optional) A callback function to handle the result or error.
    

### Using this library to perform authorization

Now, let's apply this knowledge by building a simple server. Upon successful login, the server will provide us with user information.

```javascript
const express = require("express");
const jwt = require("jsonwebtoken");
const bodyParser = require("body-parser");

const app = express();
const jwtSecret = "mySuperSecretKey"; // Secret key for JWT

// Middleware to parse JSON bodies
app.use(bodyParser.json());

// In-memory user database
const USERS = [
  {
    username: "john.doe@gmail.com",
    password: "password123",
    name: "John Doe",
    age: 30,
    city: "New York",
  },
  {
    username: "jane.smith@gmail.com",
    password: "mysecretpassword",
    name: "Jane Smith",
    age: 25,
    city: "San Francisco",
  },
  {
    username: "sam.wilson@gmail.com",
    password: "anotherpassword",
    name: "Sam Wilson",
    age: 35,
    city: "Chicago",
  },
];

// Function to check if a user exists
function getUser(username, password) {
  return USERS.find((user) => user.username === username && user.password === password);
}

// Sign-in route
app.post("/signin", (req, res) => {
  const { username, password } = req.body;

  // Check if username and password are provided
  if (!username || !password) {
    return res.status(400).json({ msg: "Username and password are required" });
  }

  const user = getUser(username, password);

  if (!user) {
    return res.status(403).json({ msg: "Invalid username or password" });
  }

  // Generate JWT
  const token = jwt.sign({ username: user.username }, jwtSecret, { expiresIn: '1h' });
  res.json({ token });
});

// Protected route
app.get("/profile", (req, res) => {
  const authHeader = req.headers.authorization;
  if (!authHeader) {
    return res.status(401).json({ msg: "Authorization header is required" });
  }

  const token = authHeader.split(' ')[1];
  if (!token) {
    return res.status(401).json({ msg: "Token is required" });
  }

  try {
    const decoded = jwt.verify(token, jwtSecret);
    const user = USERS.find((user) => user.username === decoded.username);

    if (!user) {
      return res.status(404).json({ msg: "User not found" });
    }

    // Return user's profile information
    res.json({
      msg: `Welcome, ${user.name}!`,
      profile: {
        username: user.username,
        name: user.name,
        age: user.age,
        city: user.city,
      },
    });
  } catch (err) {
    res.status(403).json({ msg: "Invalid or expired token" });
  }
});

// Start the server
app.listen(3000, () => {
  console.log("Server is running on http://localhost:3000");
});
```

### Testing with Postman

Let's test the working of our server with Postman. Below are the steps that you can follow to set-up the environment and test it on your machine.

#### Step 1: Start the Server

Type out the code above into your editor. Ensure your server is running by executing:

```bash
node app.js
```

#### Step 2: Sign-In Request

1. **Create a New Request** in Postman.
    
2. **Set the Request Type** to `POST`.
    
3. **Set the Request URL** to [`http://localhost:3000/signin`](http://localhost:3000/signin).
    
4. **Set the Headers**:
    
    * Add `Content-Type`: `application/json`.
        
5. **Set the Request Body**:
    
    * Select `raw` and choose `JSON` from the dropdown.
        
    * Enter the following JSON:
        
        ```json
        {
          "username": "john.doe@gmail.com",
          "password": "password123"
        }
        ```
        
6. **Send the Request**: Click `Send`.
    
7. **Receive the Response**: You should get a JSON response containing the JWT token:
    
    ```json
    {
      "token": "your-jwt-token-here"
    }
    ```
    

#### Step 3: Access Protected Route

1. **Create a New Request** in Postman.
    
2. **Set the Request Type** to `GET`.
    
3. **Set the Request URL** to [`http://localhost:3000/profile`](http://localhost:3000/profile).
    
4. **Set the Headers**:
    
    * Add `Content-Type`: `application/json`.
        
    * Add `Authorization`: `Bearer your-jwt-token-here` (replace `your-jwt-token-here` with the actual token you received from the sign-in response).
        
5. **Send the Request**: Click `Send`.
    
6. **Receive the Response**: If the token is valid, you should get the user's profile information and a welcome message. If the token is invalid, you should get a response indicating the token is invalid or expired.
    

### Conclusion

In this way, we can create a basic user authentication system using JWTs. With minor modifications, the above code can be expanded to include a registration functionality and specific resource access for authenticated users, which I will cover in a future article. I encourage you to experiment further with JWTs and the `jsonwebtoken` library, as this guide only scratches the surface. Visit [jwt.io](http://jwt.io) to explore more and see what you can achieve.