---
title: "Build an App"
full_title: "Lesson 5.1: Build an App"
weight: 6
date: 2020-10-23T1:00:00-00:96
draft: true
---

At this point, we've put all the pieces together. Now let's keep going and build an app from the ground up.

We're goign to install node on our VM first so we can build locally and then move to a container. IF you're working on your local machine, you will need to have node/npm.
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.36.0/install.sh | bash
. ~/.nvm/nvm.sh
nvm install node
node --version
nvm install 12.14.1
node --version
```

```bash
mkdir node_project
cd node_project
npm init -y
```

Create a new file package.json and run install
```json
{
  "name": "nodejs-image-demo",
  "version": "1.0.0",
  "description": "nodejs image demo",
  "author": "Sammy the Shark <sammy@example.com>",
  "license": "MIT",
  "main": "app.js",
  "keywords": [
    "nodejs",
    "bootstrap",
    "express"
  ],
  "dependencies": {
    "express": "^4.16.8"
  }
}
```
```bash
npm install
```

Let's create and start building up app.js
```javascript
const express = require('express');
const app = express();
const router = express.Router();

const path = __dirname + '/views/';
const port = 8080;
```

Add the routes:
```javascript
router.use(function (req,res,next) {
  console.log('/' + req.method);
  next();
});

router.get('/', function(req,res){
  res.sendFile(path + 'index.html');
});

router.get('/sharks', function(req,res){
  res.sendFile(path + 'sharks.html');
});
```

Mount the router middleware / static assets and then start the app on port 8080:
```javascript
app.use(express.static(path));
app.use('/', router);

app.listen(port, function () {
  console.log('Listening on port 8080!')
})
```

Create the index view
```bash
mkdir views
code views/index.html
```

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <title>About Sharks</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <link href="css/styles.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css?family=Merriweather:400,700" rel="stylesheet" type="text/css">
</head>

<body>
    <nav class="navbar navbar-dark bg-dark navbar-static-top navbar-expand-md">
        <div class="container">
            <button type="button" class="navbar-toggler collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false"> <span class="sr-only">Toggle navigation</span>
            </button> <a class="navbar-brand" href="#">Everything Sharks</a>
            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                <ul class="nav navbar-nav mr-auto">
                    <li class="active nav-item"><a href="/" class="nav-link">Home</a>
                    </li>
                    <li class="nav-item"><a href="/sharks" class="nav-link">Sharks</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="jumbotron">
        <div class="container">
            <h1>Want to Learn About Sharks?</h1>
            <p>Are you ready to learn about sharks?</p>
            <br>
            <p><a class="btn btn-primary btn-lg" href="/sharks" role="button">Get Shark Info</a>
            </p>
        </div>
    </div>
    <div class="container">
        <div class="row">
            <div class="col-lg-6">
                <h3>Not all sharks are alike</h3>
                <p>Though some are dangerous, sharks generally do not attack humans. Out of the 500 species known to researchers, only 30 have been known to attack humans.
                </p>
            </div>
            <div class="col-lg-6">
                <h3>Sharks are ancient</h3>
                <p>There is evidence to suggest that sharks lived up to 400 million years ago.
                </p>
            </div>
        </div>
    </div>
</body>

</html>
```

Create the sharks view
```bash
code views/sharks.html
```

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <title>About Sharks</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <link href="css/styles.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css?family=Merriweather:400,700" rel="stylesheet" type="text/css">
</head>
<nav class="navbar navbar-dark bg-dark navbar-static-top navbar-expand-md">
    <div class="container">
        <button type="button" class="navbar-toggler collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false"> <span class="sr-only">Toggle navigation</span>
        </button> <a class="navbar-brand" href="/">Everything Sharks</a>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav mr-auto">
                <li class="nav-item"><a href="/" class="nav-link">Home</a>
                </li>
                <li class="active nav-item"><a href="/sharks" class="nav-link">Sharks</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
<div class="jumbotron text-center">
    <h1>Shark Info</h1>
</div>
<div class="container">
    <div class="row">
        <div class="col-lg-6">
            <p>
                <div class="caption">Some sharks are known to be dangerous to humans, though many more are not. The sawshark, for example, is not considered a threat to humans.
                </div>
                <img src="https://assets.digitalocean.com/articles/docker_node_image/sawshark.jpg" alt="Sawshark">
            </p>
        </div>
        <div class="col-lg-6">
            <p>
                <div class="caption">Other sharks are known to be friendly and welcoming!</div>
                <img src="https://assets.digitalocean.com/articles/docker_node_image/sammy.png" alt="Sammy the Shark">
            </p>
        </div>
    </div>
</div>

</html>
```

Create a stylesheet
```bash
mkdir views/css
code views/css/styles.css
```

```css
.navbar {
    margin-bottom: 0;
}

body {
    background: #020A1B;
    color: #ffffff;
    font-family: 'Merriweather', sans-serif;
}

h1,
h2 {
    font-weight: bold;
}

p {
    font-size: 16px;
    color: #ffffff;
}

.jumbotron {
    background: #0048CD;
    color: white;
    text-align: center;
}

.jumbotron p {
    color: white;
    font-size: 26px;
}

.btn-primary {
    color: #fff;
    text-color: #000000;
    border-color: white;
    margin-bottom: 5px;
}

img,
video,
audio {
    margin-top: 20px;
    max-width: 80%;
}

div.caption: {
    float: left;
    clear: both;
}
```

```bash
tree -I node_modules
```
```
.
├── app.js
├── package-lock.json
├── package.json
└── views
    ├── css
    │   └── styles.cssno
    ├── index.html
    └── sharks.html

2 directories, 6 files
```

Run the app natively
```bash
cd ~/node_project
node app.js
```

Running "application" on the local machine
!["Sharks"](/getting_started_with_containerization/images/lesson6/sharks.png)

Now let's create a container to run this application
```bash
FROM alpine:3.12.1
RUN apk add --update nodejs npm
RUN addgroup -S node && adduser -S node -G node
USER node
RUN mkdir /home/node/src
WORKDIR /home/node/src 
COPY --chown=node:node package-lock.json package.json ./
RUN npm ci 
COPY --chown=node:node . .
CMD [ "node", "app.js" ]
```

NOTE: I'm using `npm ci`, not npm install here. `npm ci`. If you use npm ci, you’ll get reliable builds. This is useful when you’re running in a continuous integration tool like Jenkins or GitLab CI.
ref: (https://medium.com/better-programming/npm-ci-vs-npm-install-which-should-you-use-in-your-node-js-projects-51e07cb71e26)

Before building the image, let's add a `.dockerignore` file. If works exactly like .gitignore
```txt
node_modules
npm-debug.log
Dockerfile
.dockerignore
```

Let's build the image
```bash
docker build -t <your_dockerhub_username>/nodejs-image-demo .
```

Let's run the image
```bash
docker run --name nodejs-image-demo -p 80:8080 -d wesreisz/nodejs-image-demo
```

Now that's add nodemon and mount the source code through a volume mount
```bash
docker run --name nodejs-image-demo --mount type=bind,source=${PWD},target=/home/node/src  -p 80:8080 -d wesreisz/nodejs-image-demo
```

Because we are using bind mounts, I can update the code directly that we built into the container. I can use something like nodemon or other watch tools to force a rebuild.