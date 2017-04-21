# JWT Clientside

We're going to use **Local Storage** to store our JWTs and make a single-page application (SPA) using AJAX.

> **Note: Send over the starter-code**

```
├── config
│   ├── apiRoutes.js
│   ├── config.js
│   └── webRoutes.js
├── controllers
│   ├── authentications.js
│   ├── statics.js
│   └── users.js
├── gulpfile.js
├── index.html
├── index.js
├── models
│   └── user.js
├── package.json
└── src
    ├── css
    │   └── style.css
    └── js
        └── app.js

6 directories, 13 files
```

We have done things a little differently and combined the `api` and the `client` apps into one.

This will make it much easier to manage both the api and client files as they are located in the same directory.

Let's start up the app by running.

```bash
$ npm install
$ gulp
```

> **Note:
Make sure you are running `mongod` in another terminal session.**

First, we need to set up a little menu to help us navigate through the app easier, we are going to be using bootstrap 4 for this lesson.

```html
<nav class="navbar navbar-light bg-faded">
  <div class="container">
    <a class="navbar-brand" href="#">WDI Yearbook</a>
    <button class="navbar-toggler hidden-sm-up" type="button" data-toggle="collapse" data-target="#exCollapsingNavbar2" aria-controls="exCollapsingNavbar2" aria-expanded="false" aria-label="Toggle navigation">
      &#9776;
    </button>
    <div class="collapse navbar-toggleable-xs" id="exCollapsingNavbar2">
      <ul class="nav navbar-nav">
        <li class="nav-item active">
          <a class="nav-link" href="#">Users <span class="sr-only">(current)</span></a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Register</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Login</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Logout</a>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

If you refresh the page you should see something like this.

<img width="1280" alt="screen shot 2017-01-06 at 14 47 24" src="https://cloud.githubusercontent.com/assets/11501555/21721210/3a12f41a-d41f-11e6-91c3-356c8c4565e2.png">

Great, let's now start writing some javascript

> **Note: Rememeber that we DON'T write any code inside the public directory**

Inside `src/js/app.js`:

```js
const App = App || {};

App.init = function() {
  console.log('loaded');
}

$(App.init);
```

We want the value of `this` to be the object that we have just created, so we need to use a `.bind` to do this.

```js
$(App.init.bind(App));
```

## Register form

Now, we want to show some kind of form so that the user can regiter with some data.

Let's add the class of `register` to the register link inside `index.html`.

```html
<li class="nav-item">
  <a class="nav-link register" href="#">Register</a>
</li>
```

Now, let's add a click event on this link.

```js
App.init = function() {
  $('.register').on('click', function() {
    console.log('You clicked register');
  });
};
```

Instead of using an anonymous function here, Ideally we want to use a named one because we might use it again sometime later on.

```js
App.register = function(e) {
  e.preventDefault();
  console.log('You clicked register');
};
```

and then call the named function as the callback to the click event.

```js
App.init = function() {
  $('.register').on('click', this.register.bind(this));
};
```

> **Note: We need to bind the value of `this` to the `App` object again because of the default behaviour of a click event**

Now, let's inject a `<form>` into the `<main>` whenever this link has been clicked.

Firstly, because we are going to be interacting with the `<main>` multiple times, lets add a variable inside our code for it.

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
};
```

Now, inside the callback function for the click event, we want to append a form into the main like this.

```js
App.register = function(e) {
  e.preventDefault();
  this.$main.html(`
  <h2>Register</h2>
  <form method="post" action="/register">
    <div class="form-group">
      <input class="form-control" type="text" name="user[username]" placeholder="Username">
    </div>
    <div class="form-group">
      <input class="form-control" type="email" name="user[email]" placeholder="Email">
    </div>
    <div class="form-group">
      <input class="form-control" type="password" name="user[password]" placeholder="Password">
    </div>
    <div class="form-group">
      <input class="form-control" type="password" name="user[passwordConfirmation]" placeholder="Password Confirmation">
    </div>
    <input class="btn btn-primary" type="submit" value="Register">
  </form>
  `);
};
```

When we click on the register button in the navbar, we should we something like this.

<img width="1278" alt="screen shot 2017-01-06 at 15 10 13" src="https://cloud.githubusercontent.com/assets/11501555/21721937/52e2f1ae-d422-11e6-95c1-41b544c0a610.png">

Cool, so we've added a register form, next thing we need to do is create an AJAX request when the form has been submitted so that we can POST the data to the api.

To do this, first we need to add an event listener to listen for a form submission.

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
  this.$main.on('submit', 'form', this.handleForm);
};
```

Because the form is dynamically created, we need to use event delegation to add this event listener.

Next let's make the `handleForm` callback function.

```js
App.handleForm = function(e) {
  e.preventDefault();
  console.log('form submitted');
}
```

Inside this function, we want to make an AJAX request.

Because our url to our api is going to be the same, let's create a vairable for this value.

```js
App.init = function() {
  this.$main  = $('main');
  this.apiUrl = 'http://localhost:3000/api';
  $('.register').on('click', this.register.bind(this));
  this.$main.on('submit', 'form', this.handleForm);
};
```

Now, let's write the AJAX request

```js
App.handleForm = function(e) {
  e.preventDefault();

  const url    = `${App.apiUrl}${$(this).attr('action')}`;
  const method = $(this).attr('method');
  const data   = $(this).serialize();

  $.ajax({
    url,
    method,
    data
  }).done((data) => {
    console.log(data);
  }).fail(data) => {
    console.log(data);
  });
}
```

We need to create the correct url for the register route in our api, we can use the `apiUrl` variable that we just created and the action attribute from the form to do this.

```js
const url    = `${App.apiUrl}${$(this).attr('action')}`;
```

We also want to get the method attribute because this will give us the method for the request.

```js
const method = $(this).attr('method');
```

And we need to specify some data to post to the api.

```js
const data   = $(this).serialize();
```

Great, now lets go back to the browser and register with some data and we should see the data returned inside the console. This should contain a message, a user object containing the data for the user that has just registered and a token.

<img width="1280" alt="screen shot 2017-01-06 at 15 34 50" src="https://cloud.githubusercontent.com/assets/11501555/21722730/cbb53f30-d425-11e6-985d-3a1869755d4a.png">

## Local Storage

We need a way to be able to take the token and store it somewhere so we can access it.

We are going to store this in something called `localStorage`.

Local Storage is basicly a storage located in the browser where you can store key, value pairs.

```js
App.handleForm = function(e) {
  e.preventDefault();

  const url    = `${App.apiUrl}${$(this).attr('action')}`;
  const method = $(this).attr('method');
  const data   = $(this).serialize();

  $.ajax({
    url,
    method,
    data
  }).done((data) => {
    if (data.token) App.setToken(data.token);
  }).fail(data) => {
    console.log(data);
  });
}
```

When the request has been made and is successful, inside the callback `.done` we want to use a function which will set the token into localStorage.

```js
App.setToken = function() {
  return window.localStorage.setItem('token', token);
}
```

> **Note: `.setItem()` is a method which is given to us as part of the localStorage window property, it takes two arguements, a key and a value.**

Now, refresh the browser and register with some new data, go to the application tab of the dev tools and click on local storage on the left side bar, we should see our token.

<img width="1280" alt="screen shot 2017-01-06 at 15 47 34" src="https://cloud.githubusercontent.com/assets/11501555/21723249/a734bcc4-d427-11e6-90e4-b2ad7b58defb.png">

## Users Index

Now we have the ability to register and store the token somewhere, we can now use this token to access the users index data.

Because the users index can only be viewed by someone who is logged in, we are going to have to pass the token through as part of the request.

Let's add a click event listener to the `users` link in the navbar.

```html
<li class="nav-item active">
  <a class="nav-link usersIndex" href="#">Users <span class="sr-only">(current)</span></a>
</li>
```

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
  $('.usersIndex').on('click', this.usersIndex.bind(this));
  this.$main.on('submit', 'form', this.handleForm);
};
```

```js
App.usersIndex = function(e) {
  e.preventDefault();

  const url = `${this.apiUrl}/users`;

  $.ajax({
    url
  }).done(data => {
    console.log(data);
  }).fail(data => {
    console.log(data);
  });
}
```

If we try to click the link now, we will get an error saying `unauthrozied request` this is because we have not passed the token through as part of the request.

We need to change the ajax request to send the JWT token.

Firstly, we need a function to be able to get the token from the localStorage

```js
App.getToken = function() {
  return window.localStorage.getItem('token');
}
```

> **Note: Just like `.setItem()`, `.getItem()` is also a method which is accessed through the localStorage property**

Now we can use this function to access the token value so that we can pass it through with the ajax request.

```js
App.usersIndex = function(e) {
  e.preventDefault();

  const url = `${this.apiUrl}/users`;

  $.ajax({
    url,
    beforeSend: function(xhr) {
      return xhr.setRequestHeader('Authorization', `Bearer ${App.getToken()}`);
    }
  }).done(data => {
    console.log(data);
  }).fail(data => {
    console.log(data);
  });
}
```

Great, if we refresh the browser now and click on users we should see this in the console.

<img width="1280" alt="screen shot 2017-01-06 at 16 10 43" src="https://cloud.githubusercontent.com/assets/11501555/21724083/c8706b06-d42a-11e6-8b66-a1e1de07ac46.png">

Now we are able to get the data for the users. Let's actually append the data to the page.

```js
App.usersIndex = function(e) {
  e.preventDefault();

  const url = `${this.apiUrl}/users`;

  $.ajax({
    url,
    beforeSend: function(xhr) {
      return xhr.setRequestHeader('Authorization', `Bearer ${App.getToken()}`);
    }
  }).done(data => {
    this.$main.html(`
      <div class="card-deck-wrapper">
        <div class="card-deck">
        </div>
      </div>
    `);
    const $container = this.$main.find('.card-deck');
    $.each(data.users, (i, user) => {
      $container.append(`
        <div class="card col-md-4">
         <img class="card-img-top" src="http://fillmurray.com/300/300" alt="Card image cap">
         <div class="card-block">
           <h4 class="card-title">${user.username}</h4>
           <p class="card-text">This is a longer card with supporting text below as a natural lead-in to additional content. This content is a little bit longer.</p>
           <p class="card-text"><small class="text-muted">Last updated 3 mins ago</small></p>
         </div>
       </div>`);
    });
  }).fail(data => {
    console.log(data);
  });
}
```

If everything has worked, we should see something that looks like this.

<img width="1280" alt="screen shot 2017-01-06 at 16 18 20" src="https://cloud.githubusercontent.com/assets/11501555/21724325/d5d39632-d42b-11e6-996b-0119fa177773.png">

## Dry up code

let's take some time now to dry up some of the code that we have written so far.

firstly, we want to make the function that is setting the request header a named function as we might use it again in the future.

```js
App.setRequestHeader = function(xhr) {
  return xhr.setRequestHeader('Authorization', `Bearer ${App.getToken()}`);
};
```

```js
App.usersIndex = function(e) {
  e.preventDefault();

  const url = `${this.apiUrl}/users`;

  $.ajax({
    url,
    beforeSend: this.setRequestHeader.bind(this);
    }).done(data => {
    this.$main.html(`
      <div class="card-deck-wrapper">
        <div class="card-deck">
        </div>
      </div>
    `);
    const $container = this.$main.find('.card-deck');
    $.each(data.users, (i, user) => {
      $container.append(`
        <div class="card col-md-4">
         <img class="card-img-top" src="http://fillmurray.com/300/300" alt="Card image cap">
         <div class="card-block">
           <h4 class="card-title">${user.username}</h4>
           <p class="card-text">This is a longer card with supporting text below as a natural lead-in to additional content. This content is a little bit longer.</p>
           <p class="card-text"><small class="text-muted">Last updated 3 mins ago</small></p>
         </div>
       </div>`);
    });
  }).fail(data => {
    console.log(data);
  });
}
```

Great, now let's make a function to handle any ajax request we will make.

```js
App.ajaxRequest = function(url, method, data, callback) {
  return $.ajax({
    url,
    method,
    data,
    beforeSend: this.setRequestHeader.bind(this)
    })
    .done(callback)
    .fail(data => {
      console.log(data);
    });
};
```

Now we can replace our ajax requests so that they are using the function we can just written

```js
App.handleForm = function(e) {
  e.preventDefault();

  const url    = `${App.apiUrl}${$(this).attr('action')}`;
  const method = $(this).attr('method');
  const data   = $(this).serialize();

  return App.ajaxRequest(url, method, data, data => {
    if (data.token) App.setToken(data.token);
  });
}
```

```js
App.usersIndex = function(e) {
  e.preventDefault();

  const url = `${this.apiUrl}/users`;

  return App.ajaxRequest(url, 'get', null, data => {
    this.$main.html(`
      <div class="card-deck-wrapper">
        <div class="card-deck">
        </div>
      </div>
    `);
    const $container = this.$main.find('.card-deck');
    $.each(data.users, (i, user) => {
      $container.append(`
        <div class="card col-md-4">
         <img class="card-img-top" src="http://fillmurray.com/300/300" alt="Card image cap">
         <div class="card-block">
           <h4 class="card-title">${user.username}</h4>
           <p class="card-text">This is a longer card with supporting text below as a natural lead-in to additional content. This content is a little bit longer.</p>
           <p class="card-text"><small class="text-muted">Last updated 3 mins ago</small></p>
         </div>
       </div>`);
    });
  });
};
```

Test out the app again and make sure both links are still working.

## Login form

Let's set up the login form and ajax request, this will be very similar to the register function.

```html
<li class="nav-item">
  <a class="nav-link login" href="#">Login</a>
</li>
```

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
  $('.login').on('click', this.login.bind(this));
  $('.usersIndex').on('click', this.usersIndex.bind(this));
  this.$main.on('submit', 'form', this.handleForm);
};
```

```js
App.login = function(e) {
  e.preventDefault();
  this.$main.html(`
    <h2>Login</h2>
    <form method="post" action="/login">
      <div class="form-group">
        <input class="form-control" type="email" name="email" placeholder="Email">
      </div>
      <div class="form-group">
        <input class="form-control" type="password" name="password" placeholder="Password">
      </div>
      <input class="btn btn-primary" type="submit" value="Login">
    </form>
  `);
};
```

## Logged in vs Logged out

We are now going to create two functions to determine wether someone is logged in or not.

These will allow us to alter the nav links and redirect to different functions depending on if someone is logged in.

```js
App.loggedInState = function() {
  $('.loggedIn').show();
  $('.loggedOut').hide();
};
```

```js
App.loggedOutState = function() {
  $('.loggedIn').hide();
  $('.loggedOut').show();
};
```

We can how give certain elements these classes to toggle them showing.

```html
<ul class="nav navbar-nav">
  <li class="nav-item active loggedIn">
    <a class="nav-link" href="#">Users <span class="sr-only">(current)</span></a>
  </li>
  <li class="nav-item loggedOut">
    <a class="nav-link" href="#">Register</a>
  </li>
  <li class="nav-item loggedOut">
    <a class="nav-link" href="#">Login</a>
  </li>
  <li class="nav-item loggedIn">
    <a class="nav-link" href="#">Logout</a>
  </li>
</ul>
```

In our init function, we need to check if someone is logged in or not.

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
  $('.login').on('click', this.login.bind(this));
  $('.usersIndex').on('click', this.usersIndex.bind(this));
  this.$main.on('submit', 'form', this.handleForm);

  if (this.getToken()) {
  	this.loggedInState();
  } else {
    this.loggedOutState();
  }
};
```

Now we should see the links in the navbar change depend of if the user is logged in or not.

## Logout

Finally, we need to write a function to handle the logout.

```html
<li class="nav-item loggedIn">
  <a class="nav-link logout" href="#">Logout</a>
</li>
```

```js
App.init = function() {
  this.$main = $('main');
  $('.register').on('click', this.register.bind(this));
  $('.login').on('click', this.login.bind(this));
  $('.logout').on('click', this.logout.bind(this));
  $('.usersIndex').on('click', this.usersIndex.bind(this));
  this.$main.on('submit', 'form', this.handleForm);

  if (this.getToken()) {
  	this.loggedInState();
  } else {
    this.loggedOutState();
  }
};
```

Let's create a function to clear the localStorage.

```js
App.removeToken = function() {
  return window.localStorage.clear();
};
```

Now we can call this function inside our logout function, then we can call `loggedOutState()` to trigger the correct nav links to show.

```js
App.logout = function(e) {
  e.preventDefault();

  this.removeToken();
  this.loggedOutState();
}
```

Quickly, lets just add `loggedInState()` to when someone has registered or logged in so that the correct nav links show then too.

```js
App.handleForm = function(e) {
  e.preventDefault();

  const url    = `${App.apiUrl}${$(this).attr('action')}`;
  const method = $(this).attr('method');
  const data   = $(this).serialize();

  return App.ajaxRequest(url, method, data, data => {
    if (data.token) App.setToken(data.token);
    App.loggedInState();
  });
}
```

Next, we want to change the content on the page to display the right content, e.g display the users index once a user has registered/loggedin


```js
App.loggedInState = function() {
  $('.loggedIn').show();
  $('.loggedOut').hide();
  this.usersIndex();
}
```

Because we are calling `usersIndex()` not as an event callback, `e` is going to be undefined. to fix this we need to change the use of `e` inside the function to be within a `if` block.

```js
App.usersIndex = function(e) {
  if (e) e.preventDefault();

  const url = `${this.apiUrl}/users`;

  return App.ajaxRequest(url, 'get', null, data => {
    this.$main.html(`
      <div class="card-deck-wrapper">
        <div class="card-deck">
        </div>
      </div>
    `);
    const $container = this.$main.find('.card-deck');
    $.each(data.users, (i, user) => {
      $container.append(`
        <div class="card col-md-4">
         <img class="card-img-top" src="http://fillmurray.com/300/300" alt="Card image cap">
         <div class="card-block">
           <h4 class="card-title">${user.username}</h4>
           <p class="card-text">This is a longer card with supporting text below as a natural lead-in to additional content. This content is a little bit longer.</p>
           <p class="card-text"><small class="text-muted">Last updated 3 mins ago</small></p>
         </div>
       </div>`);
    });
  });
};
```

Let's do the same of the `loggedOutState()`


```js
App.loggedOutState = function() {
  $('.loggedIn').hide();
  $('.loggedOut').show();
  this.register();
}
```

```js
App.register = function(e) {
  if (e) e.preventDefault();

  this.$main.html(`
    <h2>Register</h2>
    <form method="post" action="/register">
      <div class="form-group">
        <input class="form-control" type="text" name="user[username]" placeholder="Username">
      </div>
      <div class="form-group">
        <input class="form-control" type="email" name="user[email]" placeholder="Email">
      </div>
      <div class="form-group">
        <input class="form-control" type="password" name="user[password]" placeholder="Password">
      </div>
      <div class="form-group">
        <input class="form-control" type="password" name="user[passwordConfirmation]" placeholder="Password Confirmation">
      </div>
      <input class="btn btn-primary" type="submit" value="Register">
    </form>
  `);
};
```

Great! Now hopefully we should now have a fully working frontend for authentication.
