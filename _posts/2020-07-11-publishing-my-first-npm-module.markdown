---
layout: post
title: Publishing My First npm Module
date: 2020-07-11 14:41:20 +0000
description: why and how I published ts-express-cli # Add post description (optional)
img: npm.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [TypeScript, Node.js, NPM, Library]
---

The library in focus here is the [ts-express-cli](https://www.npmjs.com/package/ts-express-cli); A minimal TypeScript express server project generator, which I released  a couple of weeks ago. My love for TypeScript is never hidden and I always use it wherever possible.

# The problem
It's common knowledge it is very easy and fast to experiment with ideas using JS/TS than other languages, which I do a lot. Every time I setup an express server I must include the `express-types`, `typescript`, `ts-types` etc to configure express to work well with TypeScript. 

This repetitive tasks is almost always the same for all projects expect the few specific libraries I must include for some special projects. The few tools I found online either does not use express to generate the server or it included a lot of libraries for me out of the box which I don't need and will not use ever. 

So I decided to create the [ts-express-cli](https://www.npmjs.com/package/ts-express-cli) which only contains the core basic libraries need to get it running.

# The Solution

I wish a could say it took me longer to complete it but it only took me about an hour or two to complete the whole development. I built it on top of  a couple of libraries:

* *Chalk:* To get colorful displays in the console.
* *figlet:* to get the big & bright ts-express-cli banner on the console.
* *Commander:* to define it as a command on the computer.
* *fs-extra:* to create files and directory on the computer.
* *prompts:* to take input from the user on the terminal for the project name, author and license.

The pull code for the library is available on GitHub [https://github.com/idawud/ts-express-cli](https://github.com/idawud/ts-express-cli)

# Usage

To install it run `npm i -g ts-express-cli` then run `ts-express-cli` on your terminal to create project.

![ts-express-cli]({{site.baseurl}}/assets/img/support/ts-express-cli.png)

This will create a hello world index route in `src/app.ts`,
`.gitignore`, `tsconfig.json`, `package.json` and `tslint.json`. You will only have to run `npm install` to install the dependencies.

The `package.json` contains a start, build and dev script. The `build script` transpile the Typescript into JavaScript es6 in a `dist/` directory. The `start script `builds and run the code while the `dev script` runs it on nodemon which restart every time a file change occurs typically during development.

# Conclusion
This simple tools has saved me a lot of time start new TS projects. In the first week of publishing it, it about 200 downloads without me advertising it to any developer group or persons. This just tells me how much others needed a tool like.