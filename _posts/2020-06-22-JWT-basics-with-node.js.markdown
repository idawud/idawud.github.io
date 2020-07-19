---
layout: post
title: JWT With Node.js
date: 2020-06-22 14:41:20 +0000
description: How to create a authentication web service with JSON Web Token  # Add post description (optional)
img: jwt.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Authentication, Node.js, JWT, Docker, API]
---

JWT (JSON Web Token) is a means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature (JWS) and/or encrypted using JSON Web Encryption (JWE).

The tokens are signed either using a private secret or a public/private key. For example, a server could generate a token that has the claim "logged in as admin" and provide that to a client. The client could then use that token to prove that it is logged in as admin. The tokens can be signed by one party's private key (usually the server's) so that party can subsequently verify the token is legitimate. If the other party, by some suitable and trustworthy means, is in possession of the corresponding public key, they too are able to verify the token's legitimacy.

# Setup project
I like to create my Node.js projects with TypeScript and to create a minimal node.js express server I use my own cli tool  [ts-express-cli](https://www.npmjs.com/package/ts-express-cli) to generate the project. The only thing I have to provide is the project name, the author name and an optional license type for the package.json.

This will create a project directory with `src/app.ts`, `tslint.json` and `tsconfig.json`. Run `npm install` in the generated project to install the dependencies. Since we are going to work with jsonwebtoken we have to include it's library and it type since we are working with TypeScript, to do that run `npm i jsonwebtoken --save` and `npm i -D @types/jsonwebtoken`. These are all the dependencies we need for this tutorial.

# Middleware
We are going to create two routes; one to generate the token which is typically part of the login system and the other to verify we are dealing with a valid token before access to resources is granted.
```ts
app.post('/post', verifyToken );
app.post('/login', generateToken);
```
Before we go any further let's import jwt into our src/app.ts file 
```ts
import jwt from 'jsonwebtoken';
```

# Generate a Token
We start by creating a normal express middleware which takes `Request, Response, NextFunction` as parameters.
```ts
const generateToken = (req: Request, res: Response, next: NextFunction) => {
	const user = { name: 'idawud', email: 'ismaildawud96@gmail.com'};

	jwt.sign({user}, 'secretKey', {expiresIn: '60s'}, (err: Error | null, token: string | undefined) => {
		if ( err != null){
			res.sendStatus(403);
		} else {
			res.json(token);
		}
	});
}
```
We will typically get the user from the database or through an openID Connect but for the sake of this tutorial we'll create a hard-coded dummy user. To generate the token the `sign` function is used this takes the user, secret key, optional expiry duration and a callback to handle the generated token or error.

What we have above uses the dummy user with the secret key as `secretKey` and expiry duration of 60 seconds. If an error occurs the a 403 response is returned else the token string is return with status of 200.

Using either POSTMAN, curl or any HTTP Client make a post request to the server at `http://localhost:5000/login`.  If everything goes okay you'll get a token string response back with status code 200.
```
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7Im5hbWUiOiJpZGF3dWQiLCJlbWFpbCI6ImlzbWFpbGRhd3VkOTZAZ21haWwuY29tIn0sImlhdCI6MTU5NTEzODEwNCwiZXhwIjoxNTk1MTM4MTY0fQ.A0Q2qT7Kx2aaUwzUUZTfaF-KA3oClzFg2qK89RzQe8A"
```

# Verify a Token
To verify our token we'll have to include the in our `Authorization` header for all protected routes, which will in the format `Bearer <full-token-string>`.

Thus in our verification middleware we first check if an Authorization Header is provided on the request. If it's not provided a 403 status is returned immediately else split the Authorization Header string by a space and get the 2nd string which is the token.
```ts
const verifyToken = (req: Request, res: Response) => {
	const bearerHeader: string | undefined = req.headers.authorization;
	if ( bearerHeader === undefined){
        console.error(`Error :: No Authorization Header`);
        res.sendStatus(403);
	} else {
        const bearer: string[] = bearerHeader.split(' ');
        const token: string = bearer[1];
        jwt.verify(token, 'secretKey', (err: Error | null, data: object | undefined)  => {
            if ( err != null){
                console.error(`Error :: ${err.message}`);
                res.sendStatus(403);
            } else {
                res.json({ data });
            }
        });
	}
}
```
To verify the token the `verify` function from the jwt library is used which takes the token string, the secret key used to create the token and a callback to handle the data or error. On our callback as usual if there is an error a 403 is returned else we return the user information encoded in the json token.

Using your favorite HTTP Client make a post request to the server at `http://localhost:5000/post`, remember to add the Authorization header, e.g.
```
Authorization : "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7Im5hbWUiOiJpZGF3dWQiLCJlbWFpbCI6ImlzbWFpbGRhd3VkOTZAZ21haWwuY29tIn0sImlhdCI6MTU5NTEzODEwNCwiZXhwIjoxNTk1MTM4MTY0fQ.A0Q2qT7Kx2aaUwzUUZTfaF-KA3oClzFg2qK89RzQe8A"
```
If everything goes okay you'll get the user object response back with status code 200.
```json
{
    "data": {
        "user": {
            "name": "idawud",
            "email": "ismaildawud96@gmail.com"
        },
        "iat": 1595139394,
        "exp": 1595139454
    }
}
```
The data contains the user, the timestamp issued at the the timestamp to expire.

# Conclusion
This short article we looked at how to generate a JSON Web Token and also saw how to verify our token string to protect routes and resources on our web service.