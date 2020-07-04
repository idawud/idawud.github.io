---
layout: post
title: Caching With Redis In Node.js
date: 2020-06-04 14:41:20 +0000
description: How to run mongoDB in a docker container and bind the volume to the system's volume  # Add post description (optional)
img: redis_node.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Redis, Node.js, Cache, Docker, API]
---
Caching (pronounced “cashing”) is the process of storing data in a cache. A cache is a temporary storage area. For example, the files you automatically request by looking at a Web page are stored on your hard disk in a cache subdirectory under the directory for your browser.Web caches reduce the amount of information that needs to be transmitted across the network, as information previously stored in the cache can often be re-used. One such tool for caching is **[Redis](https://redis.io/)** _in-memory data structure store, used as a cache, database, and a message broker_.

In this post we'll look at how to implement a cache with redis in Node.js. by caching data from the GitHub API.

### Setup Redis locally
If you are familiar with docker then this is the easiest way to run redis locally. run
`docker run --name redis -p 6379:6379 -d redis`, this will pull the redis image if not locally available run it and publish it on port `6379` the default redis port.

Alternatively install redis by following this post -> [Installing Redis](https://redis.io/topics/quickstart)

### Setup new Node project
Create a new empty directory in your development environment and run `npm init -y`, this will generate a package.json file. Now create a file `index.js` this is all we need.

#### Install dependencies 
Run  `npm i redis express node-fetch --save`, this will install 
* redis -> node.js redis client library.
* express -> minimalist web framework for node.js.
* node-fetch -> to make http requests.

Then run `npm i -D nodemon custom-env` to install these devDependencies 
* nodemon -> is a tool that helps develop node.js based applications by automatically restarting the node application when file changes in the directory are detected.
* custom-env -> is to make development more feasible by allowing multiple .env configurations for different environments. 

The next thing is to add a start script to our `package.json` file. Update the script to:
```json
"scripts": {
    "start": "nodemon index.js"
  }
```
This will allow us to start our application by running `npm start` and restart automatically whenever files in the project changes.

#### environment
In our application will listen to the app port and the redis port from our environment variable. custom-env dependency we add makes this easier. create a file `.env.development` the `development` here is our environment. add this to the file created 

```env
APP_PORT=8080
REDIS_PORT=6379
```

### Main application index.js
In step[1] we include libraries needed. In using the custom-env we can have multiple environment , so adding `env('development')` ensures we use the development environment we created.
In step[2] we read the ports from our environment and in In step[3] we create an instance of express our webserver and redis client.

Let's skip step[4] and step[5] for now. we'll come back to them later.

In step[6] we create our route which will be available on `host:port/repo/{username}`, where username is the github username we want to fetch.
Lastly step[7] starts our web server and listens on the port specified in our environment.

```js
// Include libraries step[1]
const express = require('express');
const fetch = require('node-fetch');
const redis = require('redis');
require('custom-env').env('development');

// read .env ports step[2]
const APP_PORT = process.env.APP_PORT || 3000;
const REDIS_PORT = process.env.REDIS_PORT || 6379;

// create instances step[3]
const app = express();
const client = redis.createClient(REDIS_PORT);

// Get repo info from GitHub step[4]
const getNumberOfRepo = async (req, res, next) => { }

// cache Middleware step[5]
const cache =  (req, res, next) => { }

// create a route step[6]
app.get('/repo/:username', cache, getNumberOfRepo);
 
// listen web server on port step[7]
app.listen(APP_PORT, () => {
    console.log(`App running on http://localhost:${APP_PORT}`)
});
```

#### Step[4] implementation
This step contains the main app login to fetch from github. Because we make a network call here we mark this function here as async. We wrap this function in a try/catch so that when an error occurs we can return a 500 status response. In the main logic we get the username param from the request and concat to the github base api url to fetch the  data and convert it to json. Check are make to ensure the user exist if yes the repo number is stored in the redis cache with the username as key with an expiry duration of 1 hour and then returned as json else a json error is returned.
```js
const getNumberOfRepo = async (req, res, next) => {
    try {
        console.log('Fetching data');
        // extract params here /params/:username
        const {username} = req.params; 
        
        // fetch data from github api
        const response = await fetch(`https://api.github.com/users/${username}`);
        const data = await response.json(); // convert to json

        // make sure the username exists
        if ( data.message !== 'Not Found'){
            // cache data here
            // setex params key=username, expiry duration in seconds = 3600 == 1hr, value=data.public_repos
            client.setex(username, 3600, data.public_repos);
            // return data
            res.json( { username: username, number_of_repos: data.public_repos });
        } else {
            res.status(404).json({ error: `No GitHub user ${username} Found` });
        }
        
        
    } catch (err) {
        console.error(err);
        res.status(500);
    }
}
```

#### Step[5] implementation
When we declare our route in step[6], a request first passes through the cache function before the getNumberOfRepo function. So it means we first check in our cache before we make the call. Thus if the cache contains a value for the username we return immediately else we run the next() which executes the 
getNumberOfRepo function you saw above.

```js
// cache Middleware
const cache =  (req, res, next) => {
    // get username from params
    const {username} = req.params; 

    // fetch stored value for key {username}
    client.get(username, (err, data) => {
        if (err) throw err;

        if( data !== null ){ // return from cache i.e. it's already available
            console.log('got data from cache');
            // return data
            res.json( {  username: username, number_of_repos: data });
        }else { 
            next(); // fetch from github server [getNumberOfRepo]
        }
    })
}
```

### Running the application
Run `npm start`, now open 'http://localhost:8080/repo/idawud' in your browser or curl
`curl -v -h http://localhost:8080/repo/idawud`. Should replace the port number if it was changed and you and use any github username instead of mine.

This will return a json like this if it's a valid github user
```json
{"username":"idawud","number_of_repos":64}
```
else we get a response like this, e.g. trying to access 'http://localhost:8080/repo/idawud255jhjh'
```json
{"error":"No GitHub user idawud255jhjh Found"}
```

### The effect of caching
When ever we hit the route for the first time it takes about 950ms to completely respond to the request. But on subsequent hits it takes only 25ms to respond. This is because on the subsequent requests we don't actually fetch the data again from github but from the redis storage thus making it faster.

### Conclusion
In this post we looked at what is caching and how to implement caching in a node.js web application with redis and how it improves the latency request onto the application.