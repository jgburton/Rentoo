---
title: Express Authentication JWT
type: Lesson
duration: "1:25"
creator:
    name: Alex Chin
    city: London
competencies: Server Applications
---

# Express Authentication JWT

### Objectives

- Understand why authentication tokens are commonly used when interacting with APIs
- Add a token strategy to an application
- Authenticate a user based on their token

### Preparation

- Understand how to limit JSON
- Build a basic Express app
- Understand foundational concepts in authentication & encryption

## Tokens, The Basics - Intro (10 mins)

When building APIs, authentication is crucial. APIs often give access to private, sometimes sensitive information, and we do not want to be responsible for secrets falling into the wrong hands. It's hard to be too careful, so today, we're going to learn a way to control access to an API that is both simple and secure.

The technique we're going to use today revolves around **tokens**. Tokens are, at their simplest, a unique string that is usually auto-generated. It needs to be long & complex enough that a human would never guess it, and unique enough that only one user in the database can have any particular one.

If we trust that we've designed it that way, then we only have to use a single string of characters to determine both who a user is claiming to be in our database and that they are who they say they are.

### Kicking it up a notch

That's the overall gist of what tokens do, but today we're going to use a specific type of token. It's a fairly new type of token, that's becoming widely used and trusted in web applications, and it's called a **JSON Web Token** or JWT (pronounced `jot`, if you can believe that).

It is the same idea – a single string of characters to authenticate – but this token isn't just _random_ characters, it's a string of characters that's built by encrypting actual information.

You can play with encoding/decoding the data over at their site as an example. Head on over to [jwt.io](http://jwt.io/#debugger) and see what I mean:

<img width="750" alt="JWTs" src="https://cloud.githubusercontent.com/assets/25366/9151601/2e3baf1a-3dbc-11e5-90f6-b22cda07a077.png">

#### Just like cookies, mmmm....

In the example above, you'll notice that there are 3 parts. The payload is the one we care the most about, and it holds whatever data we decide to put in there. It's very much like a cookie; we put as few things in there as possible – just the pieces we really need.

Applications can save a JWT somewhere on a user's computer, just like a cookie. Because JWTs can be encrypted into a single string, we can _also_ send it over HTTP really, really easily. Which means it'll work in any server/client scenario you can imagine. Quite nice.

## Sessions vs JWTS - Intro (5 mins)

One of the benefits of JWTs over traditional sessions is that as your website grows and you need to use multiple servers - you don't need to have a complex session storage system.

To understand this a bit more and the difference between sessions and jwts, let's think about this story:

### The Members Club

#### Sessions

I want to join a members club. I go up to the club and register. When I register, they give me a special **key** (a session key is normally stored in a cookie). Whenever I go into the club, I give the receptionist my key and they use it to open a special storage locker behind the desk (session storage). Inside the locker is some basic information about me which they can use to identify me as a real user. 

The problem with this is that as the club grows, it needs to have more and more storage space for the lockers. This is made even harder by they fact that if the club needs to open another location (new server location or just another server) in a different country, the lockers are only at one club!

The important thing to note is that without the locker, the key is actually useless. Also that the key itself does not contain any user information. It is like a key to a value in a hash. They work as a pair.

#### JWTs

The alternative solution is that a person registers to a club. When they register to the club, instead of a key the club issues them a business card with a secret password on it. 

The business card might contain some basic information about the user, their first and last name and their membership number - nothing valuable though.

When going to the club, the user presents their card and the club checks the secret password. The club uses a special phrase to check whether the secret password on the card is correct. If it is, then the user is allowed in.

Now the club doesn't need any locker storage and the only thing it needs to share between various clubs is the secret passphrase in order to decode secret passwords.

## Starter Express App - Intro (5 mins)

We're going to add JWT authentication to an existing Express app that has authentication endpoints for `/api/login` and `/api/register`.

```bash
.
├── app.js
├── config
│   ├── config.js
│   └── routes.js
├── controllers
│   ├── authentications.js
│   └── users.js
├── models
│   └── user.js
└── package.json

3 directories, 7 files
```

In this app, we have an API with these routes:
 
- **LOGIN** `POST /api/login` 
- **REGISTER** `POST /api/register`
- **USERS INDEX** `GET /api/users`
- **USERS SHOW** `GET /api/users/:id`
- **USERS UPDATE** `PUT/PATCH /api/users/:id`
- **USERS DELETE** `DELETE /api/users/:id`

### Test the API

Let's install all of the packages using `npm install`. Then let's run the app with `nodemon app`.

We can then make some requests using Insomnia, Postman or using cURL.

- Create a user with:

```
POST /api/register

{
  "username": "alexpchin",
  "email": "alex@alex.com",
  "password": "password",
  "passwordConfirmation": "password"
}
```

You should see:

```
{
  "success": true,
  "message": "Welcome alexpchin!",
  "user": {
    "_id": "56d8b16b2d3dccd9d67a3332",
    "username": "alexpchin"
  }
}
```

## Validator package - Intro (5 mins)

This app also has a new package called [`validator`](https://www.npmjs.com/package/validator). This adds a few more validations to Mongoose. We have used the package to check for a valid email address:

```js
userSchema
  .path('email')
  .validate(validateEmail);
  
.
.
.

function validateEmail(email) {
  if (!validator.isEmail(email)) {
    return this.invalidate('email', 'must be a valid email address');
  }
}
```

## The Main Feature, JWTs - Codealong (5 mins)

We'll have to install a couple npm modules to start working with JWTs & authenticating via tokens.

- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) a package for creating JWT tokens
- [express-jwt](https://www.npmjs.com/package/express-jwt) this package contains middleware that authenticates users that are using a JWT. 

```bash
$ npm install jsonwebtoken express-jwt --save
```

Now there are 3 things we're going to need to do:

- Send a token when a user registers or logs in
- A middleware that will check for the token
- An error handler for when there _isn't_ a token

That's it.

## Creating a token - Codealong (20 mins)

We currently have a register and login route. We're going to send a valid token back to the user when they login. 

Currently, our code in `controllers/authentications.js` we have the login action:

```js
function authenticationsLogin(req, res){
  User.findOne({ email: req.body.email }, (err, user) => {
    if (err) return res.status(500).json({ message: "Something went wrong." });
    if (!user || !user.validatePassword(req.body.password)) {
      return res.status(401).json({ message: "Unauthorized." });
    }

    return res.status(200).json({
      message: "Welcome back.",
      user
    });
  });
}
``` 

What we want to do is to create a new JWT token and send it back to the user. We're going to use the npm package `jsonwebtoken` to create a JWT token.

#### Require jsonwebtoken

First we need to require this package in the authentication controller:

```js
const jwt      = require('jsonwebtoken');
```

#### Create a secret

When you create a JSON token, you need a secret string. This string usually high-entropy string thay you use to encrypt the JWT token.

We're going to store this in our `config/config` file:

```js
module.exports = {
  port: process.env.PORT || 3000,
  db: 'mongodb://localhost/express-authentication-jwt',
  secret: process.env.SECRET || "gosh this is so secret... shhh..."
};
```

Now we can require this in our authentications controller:

```js
const config   = require('../config/config');
```

#### Create a JWT

Now, above where we return the JSON back to the user let's create a JWT token.

```js
config token = jwt.sign(user._id, config.secret, { expiresIn: 60*60*24 });
```

This syntax comes from the documentation, but basically we pass it the _payload_, aka `user._id`, and pass it that secret phrase we made earlier (so that tokens can be encrypted consistently) and lastly we give it an optional expiry time. This function then returns us a valid token.

> **Note:** We want to limit the payload to be as small as possible. Here we have just sent the `user._id`

#### Send the JWT to the user

Finally, we just send that JWT token back to the user just like we normally do with data. 

```js
return res.status(200).json({
  message: "Welcome back.",
  user,
  token
});
```

#### Test it out

Let's try our endpoint and see if we get a token back, using something like [Insomnia](http://insomnia.rest/), [Postman](https://www.getpostman.com/) or cURL:

```bash
POST /api/login

{
  "email": "alex@alex.com",
  "password": "password"
}
```

We shoud get back something like this:

```js
{
    "message": "Welcome back!",
    "user": {
        "_id": "56d8b2a271744c4ad7b56061",
        "username": "alexpchin",
        "email": "alex@alex.co.uk"
    },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJfYnNvbnR5cGUiOiJPYmplY3RJRCIsImlkIjoiVsOYwrLConF0TErDl8K1YGEiLCJpYXQiOjE0NTcwNDU2NDEsImV4cCI6MTQ1NzEzMjA0MX0.zl7tRcfEB-xSoun4vfynpIfwqW41BeZ8pqHlS13SAKk"
}
```

## Replicate for Register (5 mins)

Now return a JWT for when a user registers!

## Checking for a token - Codealong (5 mins)

Next up, we have to start restricting access. This is supremely easy since we're using `expressJWT`. It's built in.

#### Middleware

In `app.js`, far above where we require the routes `app.use("/api", routes);` we can protect out routes with:

```js
app.use('/api', expressJWT({ secret: config.secret })
```

This middleware sits between the request & your code, and because it's above any routes you have set up, it'll run first.

It uses the library and checks for a token. That's it. If there is a token, it keeps going & runs your app like normal. 

It throws the payload you embedded in the JWT, into `request.user` _for_ you! In this case it's going to be `user._id`.

So on any particular route or controller action, you should be able to say, `req.user._id` and get back the requesting users's id. You could use that for looking them up in the database, storing their information in an embedded document, whatever you need.

And hopefully it's apparent, but you can customize the URLs you need to restrict.

#### Omitting URLs

In this case, we need to omit the actions of `/api/register` and `/api/login` from being protected. 

```js
app.use('/api', expressJWT({ secret: config.secret })
  .unless({
    path: [
      { url: '/api/login', methods: ['POST'] },
      { url: '/api/register', methods: ['POST'] }
    ]
  }));
```

That's it. Now the last step.

## An error handler for when there isn't a token - Codealong (10 mins)

_Technically_, our app is good to go. If you try to access one of your users, you won't be able to. You'll see a bunch of junk that looks like this:

<img width="795" src="https://cloud.githubusercontent.com/assets/25366/9152366/3074b6be-3dda-11e5-8104-dba53428e936.png">

Lovely. So our last step is to pretty that up with a little error handling.

Just after your middleware, let's make another tiny little middleware:

```js
app.use(jwtErrorHandler);

function jwtErrorHandler(err, req, res, next){
  if (err.name !== "UnauthorizedError") return next();
  return res.status(401).json({ message: "Unauthorized request." });
}
```

In case you've never console.logged it, an error looks something like this:

```json
// example error
{
  "name": "UnauthorizedError",
  "message": "No authorization token was found",
  "code": "credentials_required",
  "status": 401,
  "inner": {
    "message": "No authorization token was found"
  }
}
```

We've added an `if` statement to look for this specific error. Finally, we've added a message and returned a 401 status code.

```js
return res.status(401).json({message: 'Unauthorized request.'});
```

Boom! Now let's see what happens when we try to access an user `http://localhost:3000/api/users/56d8b2a271744c4ad7b56061`:

<img width="416" alt="screen shot 2016-03-03 at 18 05 58" src="https://cloud.githubusercontent.com/assets/40461/13512942/bf532c3c-e16b-11e5-91c8-bc50424270f2.png">

A far more beautiful thing.

## Wait, don't leave us – how _do_ we access it? - Codealong (5 mins)

Last but not least, we need to actually access that resource!

You've got it all built, and this final piece will complete the puzzle.

**You send along your token via an Authorization header**, with a value of `"Bearer mylongtokengoesrighthere"`

```
curl http://localhost:3000/api/users/56d8b2a271744c4ad7b56061 --header "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJfYnNvbnR5cGUiOiJPYmplY3RJRCIsImlkIjoiVsOYwrLConF0TErDl8K1YGEiLCJpYXQiOjE0NTcwNDU2NDEsImV4cCI6MTQ1NzEzMjA0MX0.zl7tRcfEB-xSoun4vfynpIfwqW41BeZ8pqHlS13SAKk"
```

If you're using a tool other than CURL, look for where you can add in custom headers:

<img width="1221" alt="screen shot 2016-03-03 at 18 16 34" src="https://cloud.githubusercontent.com/assets/40461/13513003/195a1222-e16c-11e5-9ff3-b6106b005d95.png">

<img width="1356" alt="screen shot 2016-03-03 at 19 20 47" src="https://cloud.githubusercontent.com/assets/40461/13514230/19d90f24-e175-11e5-802d-6acfe4cdf577.png">

So, just like a client would have to, you'd:

1. POST to your `login` endpoint!
2. Copy that token!
3. GET to a specific users's endpoint, with an Authorization header!

And there you have it. It's really only a few lines of code we had to write, and once you combine it with bcrypt and hashed passwords, you've got yourself a secure API that can be authorized with a single string of characters.

## Conclusion (5 mins)

There is quite a few new concepts here. However, JWTs are really great and you will feel more comfortable the more you use them.

- What is a JWT? Why is useful for authorizing an API?
- How do you create a JWT in an endpoint in your Express app?
- How do you secure an endpoint using a JWT?
