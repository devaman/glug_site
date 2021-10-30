## Reactjs in Vuejs using Module Federation (inc Routing)

Hey everyone , Webpack has released some new cool feature called module federation. Module Federation allows a JavaScript application to dynamically load code from another application and  in the process, share dependencies. If an application consuming a federated module does not have a dependency needed by the federated code,  Webpack will download the missing dependency from that federated build origin.
## Usecase
Suppose there is a company xyz. It has a web application. It has features like landing page, blog, carrer page, etc and each of this page is managed by different teams. But on the company website it should load as one application. Also there can be case where carrer page is built using react js and landing page using Vue js . 
Previously we used to embed iframes in the container app (here it will be landing page). The problem with iframe is it loads all the dependencies again.
Using Micro frontend technique we can combine multiple app in one app and Module federation makes it easier
To learn more about Module federation [click here](https://medium.com/swlh/webpack-5-module-federation-a-game-changer-to-javascript-architecture-bcdd30e02669)
## What we going to do ?
We will be building a web application using Vuejs and react js . Here Vuejs will be our container app and Reactjs will be loaded in vue js. Also we will sync the routes for Vuejs and Reactjs.

## Project Structure
```
root
|
|-packages
  |-react-app
     |-src
         |-index.js
         |-bootstrap.js
         |-App.js
         |-components
     |-config
     |-public
     |-package.json
  |-vue-app
     |-src
         |-index.js
         |-bootstrap.js
         |-App.vue
         |-components
     |-config
     |-public
     |-package.json
|-package.json
```
Project is setup using lerna.

## Setting up Webpack

### remote (react-app)
We have one webpack.common.js. It contains all the rules for compiling different types of file like js,css, jpeg,svg etc
Now we have webpack.development.js. It imports the base config as well as run a dev-server and implements Module Federation.
Creating a remote
```js
new ModuleFederationPlugin({
      name: "auth",
      filename: "remoteEntry.js",
      exposes: {
        "./AuthApp": "./src/bootstrap"
      },
      shared: dependencies
    }),
```
Here we expose bootstrap file from react-app as AuthApp and build file is named as remoteEntry.js
Code on github

### host (vue-app)
Creating a Host
We have one webpack.common.js same as remote . In webpack.development.js we'll have webpack-dev-server as well as we specifies the remotes

```js
 new ModuleFederationPlugin({
      name: "container",
      remotes: {
        auth: "auth@http://localhost:8082/remoteEntry.js",
      },
      shared: dependencies
    }),
```
Thats it our webpack setup id done.
To run the application we'll run ```lerna setup``` in root. It will start both react and vue app.

### Mounting React app in Vue app
We will create a ReactComponent.vue file. Here we will import the mount function that we exposed from our react app.
```js
import { mount } from "auth/AuthApp";
```
Now in template we will create a div where we will mount our react app.
```vue
<template>
    <div id="react"></div>
</template>
```
Next we will call mount function in mounted lifecycle method of vue.
```js
mounted() {
this.initialPath = this.$route.matched[0].path;
    const { onParentNavigate } = mount(document.getElementById("react"), {
     initialPath: this.initialPath,
    //...
    });
    this.onParentNavigate = onParentNavigate;
  }
```
Thats it .... Now react will be mounted inside vue app
Now only one thing is left that is Routing

### Routing
We have to routing events
1. From react app to vue app (onNavigate)
2. From Vue app to react app (onParentNavigate)

We pass onNavigate callback function from vuejs to react js via mount function.
```js
 mounted() {
    this.initialPath = this.$route.matched[0].path;
    const { onParentNavigate } = mount(document.getElementById("react"), {
      initialPath: this.initialPath,
      onNavigate: ({ pathname: nextPathname }) => {
        let mext = this.initialPath + nextPathname;
        console.log("route from auth to container", mext, this.$route.path);
        if (this.$route.path !== mext) {
          this.iswatch = false;
          this.$router.push(mext);
        }
      },
      onSignIn: () => {
        console.log("signin");
      },
    });
```
We have a history.listen in our react app which will trigger this callback whenever react app route changes. In this callback function we will route our vue app to same sub route as react app route.

Now we need a callback function from react app also to sync the route when vue route changes.
In the above code block we can see a onParentNavigate function from mount function. Now when to trigger this function thats the question.
We will write a watcher function on $route
```js
 watch: {
    $route(to, from) {
      let innerRoute = this.getInnerRoute(to.path);
      if (this.iswatch) {
        if(innerRoute)
        this.onParentNavigate(innerRoute);
        else return true
      } else this.iswatch = true;
    },
  },
methods: {
    getInnerRoute(path) {
      let inner = path.split(this.initialPath)[1];
      return inner;
    },
  },
```
This is the way we have integrated the react app in vue app

[Github codebase](https://github.com/devaman/react-in-vue-webpack)
### Demo
![gif](https://raw.githubusercontent.com/devaman/react-in-vue-webpack/master/modulefederation.gif)