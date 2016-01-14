# twitter

[![Build Status](https://travis-ci.org/tatsuyaoiw/twitter.svg?branch=master)](https://travis-ci.org/tatsuyaoiw/twitter)

A minimal Twitter clone, built with [Node][node] and [Redis][redis].

## Live demo

https://twitterlikeapp.herokuapp.com/

## Features

The following features are currently implemented.

- Sign up and log in
- Follow and unfollow user
- Post tweet

## Data layout

### Users

User IDs are generated sequencially via increments:

```js
INCR user:ids => 123
```

Users are stored into Redis hashes, the key name of the hash representing a given user is `user:123`. Every hash has the following fields:

```
name (User name)
pass (Encripted password)
fullname (User's full name)
```

When we create a new user we do something like this, assuming the user is called "bob":

```
INCR user:ids => 123
HMSET user:123 name "bob" pass "asdfjkl;" fullname "Bob Marley"
```

### Followers and followings

There is another central need in our system, followers and followings. We use a Sorted Set to represent it, using the user ID of the following or follower user as element, and the unix time at which the relation between the users was created, as our score.

```
user:123:followers => Sorted Set of user IDs of all the followers users
user:123:followings => Sorted Set of user IDs of all the follwings users
```

We can add new followers with:

```
ZADD user:123:followers 1401267618 456 =>> Add user 456 with time 1401267618
```

### Tweets

Tweets also have sequencial IDs, generated by incrementing the following key:

```js
INCR tweet:ids => 789
```

Every tweet is stored into an hash named `tweet:789` with the following fields:

```
text (Body of the tweet)
created_at (Timestamp created)
user_id (User ID)
```

### Timeline

Another important thing we need is a place where we can add the updates to display in the user's home page. We'll need to access this data in chronological order, from the most recent update to the oldest. The [home timeline](https://dev.twitter.com/rest/reference/get/statuses/home_timeline) and [user timeline](https://dev.twitter.com/rest/reference/get/statuses/user_timeline) keys keep track of what tweets were made and in that order. We use a Sorted Set to achive this, using a unix timestamp for scoring, resulting in tweets being stored in chronological order.

```
user:123:user_timeline => Sorted Set of tweet IDs posted by the user
user:123:home_timeline => Sorted Set of tweet IDs posted by the user and the users they follow
```

For example, retrieving latest 5 tweet IDs from the user timeline is something like the following:

```
ZREVRANGE user:123:home_timeline 0 4
```

## Development

### Download project

First, clone this repository. When download finished, move into the project directy then run:

```
$ npm install
```

This will install all required dependent modules.

### Install Redis

Since this application requires Redis installed on your machine, you'll need to install it. For Mac OSX, the most easiest way would be using [Homebrew][homebrew].

```
$ brew install redis
```

Otherwise, please check [Redis's official documentaiton][redis] out.

### Start application

This application is based on [Express][express] framework. To start the application, run:

```
$ npm start
```

It will serve application at `http://localhost:3000/`.

## Deployment

For deployment, `Procfile` is packaged as a default configuration to integrate with Heroku. If you are not familiar with Heroku I would recommend you to first read through the documentation of [how to get started with Node.js on Heroku][heroku-getting-started-with-node].

The whole deployment process should be as simple as the follwing 3 steps.

Login to Heroku:

```
$ heroku login
````

Create an app on Heroku:

```
$ heroku create
```

Deploy an app:

```
$ git push heroku master
```

## References

- [antirez/retwis][retwis] - A Twitter-toy clone written in PHP and Redis
- [twissandra/twissandra][twissandra] - Twissandra is an example project, created to learn and demonstrate how to use Cassandra.

## License

MIT © Tatsuya Oiwa

[node]: https://nodejs.org/
[redis]: http://redis.io/
[homebrew]: http://brew.sh/
[express]: http://expressjs.com/
[heroku-getting-started-with-node]: https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction
[retwis]: https://github.com/antirez/retwis
[twissandra]: https://github.com/twissandra/twissandra
