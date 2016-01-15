# [Server.js](http://www.github.com/menway/serverjs)

Write server code in frontend JavaScript files, and more.

The purpose of this project is to unify all JavaScript code in frontend js files. With the support of server.js solution, frontend developers are now able to embed backend codes in fontend js files.

Thanks to node.js, JavaScript developers now can write one single language for both frontend and backend. But due to different host environments, the developers coming from frontend world usually need a lot of time to learn about backend development skills before getting their first node.js app running. And although frontend and backend codes are written in the same language, they are not organized well -  they have to be physically divided in fontend and backend files. This will bring much effort for further development, which usually involves code changes on both ends and also in their communicating interfaces.

Some existing solutions move the fontend codes to backend. For example, [Meteor](http://www.meteor.com) allows developers to combine codes for both ends in the same backend js file. Server.js does the similar thing, but on the opposite direction, which allows embedding server code in frontend js files. According to my project experiences, most functionalities of a web app is triggered by the frontend events, from end user's web browser. So I believe it's more reasonable to move code to the front, that the events get processed at the same place where they occurred.

Imagine that you are designing the login page for your web app. There's an input for username and another one for password and at the bottom a login button. Once the user clicks the login button, your ```onclick``` event handler is triggered, where you get the entered username and password. In traditional way, you have to submit the form to backend or write AJAX code to communicate with backend in order to get the register result, and you'll also need write backend code to receive and process this request and produce register result, in another backend js file. With server.js, you can now write code to directly check backend database right in your ```onclick``` event handler. How wonderful it is!

The ideal use case of server.js is that the developer only need play attention to frontend files and keep backend untouched, and everything just go well.



## Installation

``` sh
mkdir your_project
cd your_project
npm install serverjs --save
```



## Quick start

Under your project directory, create file ```app.js``` and write your app code (as easy as one line of code)

``` javascript
// app.js
(new require('serverjs')).start();
```

And then start you app with node.js.

``` sh
node app.js
```

Now the server is running at port 3000. Open your web browser and navigate to [http://localhost:3000](http://localhost:3000) you'll see a default ```Hello, World``` web page while the same message is printed in backend console.

``` sh
> node app.js
===================================================
Server.js running at port 3000 in development mode.
===================================================
Hello, World
```



## Directory structure

You may have noticed that in your project directory, a sub-directory named ```public``` is automatically created on your first run, which is the root folder for your website. The default ```index.html``` and ```index.js``` files are also created for a start (home page). You can also put your other html files, js files and all assets under this directory. Once you put an html file as well as a js file with the same file name, the js file will be automatically embedded in that html file within a ```script``` tag. This follows the principle of convention over configuration, which you'll see in many places in server.js solution.

``` sh
your_project/
    app.js
    public/
        index.html
        index.js
```



## Hot code reload

Inspired by [Meteor](http://www.meteor.com), hot code reload is a very useful feature during development process. Server.js also introduces this technology, which means if any file under ```public``` directory has been changed, the web browser will be automatically reloaded to reflect the change. You don't need to manually reload the page for every update.



## How to code

Once you place the server code directive ```'server code'``` on the first line of a function (or the second line, right after ```'use strict'```), it'll become a backend function, and you can write your backend code directly in it. To invoke this backend function from frontend, besides all original parameters for this function, just put an *Error-First Callback* at the end, as you usually do in most async function calls.

Following are several quick demos. Try to put them in your ```index.js``` file and see the results.

### Hello, world (default)

``` javascript
function hello() {
  'server code';
  console.log('Hello, world!');
}

hello();
```

### Hello, world (anonymous function)

``` javascript
(function() {
  'server code';
  console.log('Hello, world!');
})();
```

### Node version

``` javascript
function getNodeVersion(callback) {
  'server code';
  callback(null, process.version);
}

getNodeVersion(function(err, ver) {
  alert('Node version: ' + ver);
});
```

### File system

``` javascript
var readdir = function(callback) {
  'server code';
  var fs = require('fs');
  fs.readdir(__dirname, callback);
}

readdir(function(err, files) {
  if (err) {
    alert('Error!');
    return;
  }
  alert('Files:\n' + files.join('\n'));
});
```



## Template engine

Server.js uses [handlebars](http://handlebarsjs.com/) as the template engine. One of the benifits of using handlebars rather than other template engines is that it's compliant with plain html pages and requires little time to get familiar with. Following is a sample view written in handlebars. Handlebars support expressions, paths, helpers and many other useful features. For more detailed information, please refer to handlebars' official web page.

``` html
<!doctype html>
<html>
  <body>
    {{#if true}}
      Hello, world!
    {{else}}
      Never reach here.
    {{/if}}
  </body>
</html>
```



## The main function

One of the features of the template engine is that it accepts an initial object (aka context) and renders the web page using its value. In server.js you can output this initial object in the main function. As the similar definition of the main function in other programming languages, it is the entry of a web page and be executed only once. The object you return from the main function will be passed to template engine to render the web page. The main function need be named as ```main``` and contain the ```server code``` directive. Following is a sample of the main function.

``` html
<!-- index.html -->
<!doctype html>
<html>
  <body>
    Hello, {{name}}.
  </body>
</html>
```

``` javascript
// index.js
function main() {
  'server code';
  return {name: 'Menway'};
}
```



## Easy routing

Server.js automatically convert your static html file as a route by emitting its extention name. For example, you can access ```/public/login.html``` via ```http://your-domain.com/login```.

Server.js also introduces an easy way for routing system. The traditional way of routing static files requires nested directory structure, but in server.js, you can use a well defined file name to do the same thing. Just simply place a comma as the separator between route levels.

For example, if you need define route ```/users/profile```, instead of create ```/public/users/profile.html```, you can simply create ```/public/users,profile.html```, and it can be accessed via ```http://your-domain/users/profile```. Use the same file name for your js file and they will be mapped.

### Route parameters

You can also place route parameters in the file name, wrapped by parenthesis. For example, ```/public/users,profile,(username).html``` will accept ```/users/profile/alice```. And then in your main function, you can read ```username``` from input parameter.

Example:

``` javascript
// users,profile,(username).js
function main(username) {
  'server code';
  console.log(username);
}
```

Another example:

``` javascript
// categories,(cat),books,(book).js
function main(cat, book) {
  'server code';
  console.log(cat, book);
}
```

Note: the names of function parameters should be exact the same as the names in the route file name, but the order can be different.



## Shared functions

You can define a global name for your server code function by appending the name to server code directive. And then the function code can be globally shared for both frontend and backed, even in different js files. To invoke this shared function, use ```Server.call()```. For example,

``` javascript
// a.js
function add(a, b, callback) {
  'server code add';
  callback(null, a + b);
}

// b.js
function testAdd() {
  'server code';
  var a = 1234;
  var b = 5678;
  Server.call('add', a, b, function(err, result) {
  	console.log(result);
  });
}

// c.js
function onclick() {
  var a = parseFloat(document.getElementById('a').value);
  var b = parseFloat(document.getElementById('b').value);
  Server.call('add', a, b, function(err, result) {
  	alert(result);
  });
}
```

Note: you can also use dot characters (```.```) in the name of a shared function, e.g., ```'server code users.add';```.



## RESTful API

You can expose your server code function as a RESTful API by appending a desired route name and a http verb to the server code directive. The route name should begin with a slash character (```/```) and the http verb should be one of ```get```, ```post```, ```update``` and ```delete``` in lowercases. If the desired route name is omitted, the name of current route is used where the function is defined. If the http verb is omitted, ```get``` will be used by default. The values of the paramters are retrieved from the HTTP request header or body with the same names. Following is an example.

``` javascript
// register.js
function register(username, password, callback) {
  'server code /register post';
  // here you can ommit the route name and just write 'server code post' because /register is the current route
  // this function is now accessible via POST /register, with username and password in the HTTP request header or body

  // register this user
  // ...

  // callback(err, data)
  callback(null, {username: username});
  // if err doesn't exist, an HTTP 200 code is produced, otherwise, use new Error(code)
  // if data exists, it will be rendered in json format
}

function onsubmit() {
  var username = $('#username').val();
  var password = $('#password').val();
  var data = {username: username, password: password};
  $.post('/register', data, function(data) {
      alert('Welcome, ' + data.username);
  }, 'json');
}
```

Note: you can also expose a shared function as a RESTful API function. In this case, you need put shared function name, route name and verb after server code directive, e.g., ```'server code register /register post'```.



## Global variables

Following global variables are pre-defined in server.js.

| Variable Name | Scope             | Description                 |
| ------------- | ----------------- | --------------------------- |
| Server        | frontend, backend | delegate of the server      |
| Client        | backend           | delegate of the client      |
| Session       | backend           | current user's session data |
| Request       | backend           | current request data        |
| Response      | backend           | current response data       |



## Security considerations

During server code RPC, ```function``` typed paramters are not allowed to be transferred to the backend to avoid code injection.

In development mode, server codes are published to frontend for a better debugging, but in production mode, implementation details of server codes are hidden.

Also user friendly errors are disabled in production mode.



## Development mode

Development mode is enabled by default. You can change this setting to production mode by:

- environment variable in shell ```NODE_ENV=production node app.js```
- initialization option in app.js ```(new require('serverjs')).start({mode: 'production'})```

Behavior differences between development mode and production mode are listed as below.

| Feature              | Development mode | Production Mode |
| -------------------- | ---------------- | --------------- |
| hot code reload      | yes              | no              |
| routers cache        | no               | yes             |
| js file cache        | no               | yes             |
| js file minify       | no               | yes             |
| server code hiding   | no               | yes             |
| default service port | 3000             | 80              |
| html file minify     | no               | yes             |
| gzip compression     | no               | yes             |
| user friendly errors | yes              | no              |



## CoffeeScript support

Server.js is written in CoffeeScript, which is then compiled into JavaScript. I like CoffeeScript so much because it saved a lot of time during development, so I'd like to recommend you to use it in your development too. CoffeeScript is supported out of the box. To use it, instead of writing js files, 



## Third-party modules support

Just as regular node.js applications, you can install any third-party modules by ```npm install```. Take mongodb module as an example.

``` sh
> npm install mongodb --save
```

``` javascript
// db.js
function db() {
  'server code';
  var mongo = require('mongodb');
  
  // use mongo
  // ...
}
```



## License

MIT
