---
layout: post
title: "my node/React stack"
permalink: my-nodereact-stack
date: 2017-02-06 09:30:27
comments: true
description: "my node/React stack"
keywords: ""
categories:

tags:

---

# client side
http://blog.slatepeak.com/build-a-react-redux-app-with-json-web-token-jwt-authentication/

install dependencies wasn't working. Here's how I did it

    npm install --save-dev webpack -> ^2.2.1
    npm install --save-dev webpack-dev-server -> ^2.3.0
    npm install --save-dev extract-text-webpack-plugin -> fails. Wants webpack webpack@^1.9.1
    rm -rf node_modules/extract-text-webpack-plugin/
    npm install --save-dev extract-text-webpack-plugin@beta

    # you are then free to install from there.

The complete package.json looks like this

    {
      "name": "sass-client",
      "version": "1.0.0",
      "description": "",
      "main": "webpack.config.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "devDependencies": {
        "babel-core": "^6.23.1",
        "babel-loader": "^6.3.2",
        "babel-preset-es2015": "^6.22.0",
        "babel-preset-react": "^6.23.0",
        "babel-preset-stage-0": "^6.22.0",
        "css-loader": "^0.26.1",
        "extract-text-webpack-plugin": "^2.0.0-rc.3",
        "node-sass": "^4.5.0",
        "sass-loader": "^6.0.1",
        "style-loader": "^0.13.1",
        "webpack": "^2.2.1",
        "webpack-dev-server": "^2.3.0"
      }
    }
