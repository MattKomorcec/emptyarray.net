---
title: "How to use fetch() in JavaScript"
date: "2019-11-03"
---

Every good application has to talk to the outside world and not just with itself, right?

Maybe that sounds confusing, what I want to say is that at some point in your development, you will want to get data from some other resource, other than your own website/app. Let's say you're building an application that shows the current weather. Where are you going to get data for it? That's right, from an outside resource, probably some weather forecast service.

Since I'm a World of Tanks player(not really good, but I try!), I'll show you how to fetch(see what I did here?) my in game stats data and display it.

Almost all public API's(Application Programming Interface) require a key in order to use them(to prevent handling too many requests), you'll often have to register and get your application key. That is the case with World of Tanks API as well.

I already registered my application key, so I'm ready to start. If you want to register your own, click on the link and follow the instructions. https://developers.wargaming.net/reference/

Since this tutorial is mostly aimed at beginners, I won't go into too much detail, but there are a few types of requests you can make, so you'll often hear about GET, POST, PUT, DELETE requests. In this post we'll only do GET requests, since they are the simplest, but if you want to learn more, I'll leave some links at the bottom of the article.

First of all, you can use `fetch()` in many ways, the most straightforward way is to just provide it with an `url`, but if you want additional configuration, you can provide it with an additional object, in which you can set headers, method of the request, and so on.

In the spirit of good programming practices, we will split all the parts of the `url` we'll be requesting, to make it easy to follow. Our goal here is to get player stats, and to do that, we first have to search for a player by name to get player's `account_id`, and then we fetch the stats with that `account_id`. So we'll have to make 2 requests.

```js
const APPLICATION_ID = "XXXXXXXXXXXXXXXXXXXXX"; // This is your secret application key
let accountId = ""; // prepare the accountId variable
let playerStats = null; // prepare playerStats variable
let searchByNameUrl = `https://api.worldoftanks.eu/wot/account/list/?application_id=${APPLICATION_ID}&search=`;
let searchByAccountIdUrl = `https://api.worldoftanks.eu/wot/account/info/?application_id=${APPLICATION_ID}&account_id=`;
```

So, because we are good programmers, we named our application key with all uppercase to let ourselves know that it should be a constant, and also, since both requests require `application_id`, we can then use it in a template literal(hope you're familiar with template literals?).

Now, let's make our first request.

```js
const playerName = "Komah_1"; // This is my username, you can put whatever you want here
searchByNameUrl = searchByNameUrl + playerName; // append player name to the url that we will request

fetch(searchByNameUrl)
  .then(response => response.json()) // This parses the response to JSON
  .then(result => (accountId = result.data[0].account_id)); // Here we set the result to accountId variable
```

There are a few things going on here, but I hope it's easy to follow so far. I hope comments in the code are helpful, since they explain every line! If you're confused about the second `.then` and why we set `accountId = result.data[0].account_id`, just try to `console.log` it here instead of assigning. I also had to do that to see how the data looked. Don't worry, that's most often the case, as you're not sure what the server returns.

Good, now we have the `accountId` and that's all we needed to finally get all the player stats.

```js
searchByAccountIdUrl += accountId; // we can also shorten this :)
fetch(searchByAccountIdUrl)
  .then(response => response.json()) // again, convert to JSON
  .then(result => (playerStats = result.data[0])); // assign the result to playerStats variable
```

And this is the whole code so far:

```js
const APPLICATION_ID = "XXXXXXXXXXXXXXXXXXXXX"; // This is your secret application key
let accountId = ""; // prepare the accountId variable
let playerStats = null; // prepare playerStats variable
let searchByNameUrl = `https://api.worldoftanks.eu/wot/account/list/?application_id=${APPLICATION_ID}&search=`;
let searchByAccountIdUrl = `https://api.worldoftanks.eu/wot/account/info/?application_id=${APPLICATION_ID}&account_id=`;

const playerName = "Komah_1"; // This is my username, you can put whatever you want here
searchByNameUrl = searchByNameUrl + playerName; // append player name to the url that we will request

fetch(searchByNameUrl)
  .then(response => response.json()) // This parses the response to JSON
  .then(result => (accountId = result.data[0].account_id)); // Here we set the result to accountId variable

searchByAccountIdUrl += accountId; // we can write this shorter :)
fetch(searchByAccountIdUrl)
  .then(response => response.json()) // again, convert to JSON
  .then(result => (playerStats = result.data)); // assign the result to playerStats variable
```

And now if you run this, you can see that it actually didn't work, and there is an error in the console. I did this on purpose, to demonstrate asynchronous code. Can you spot and fix the error?

The problem is `accountId` variable, which is just an empty string in the second `fetch` call. Why? Because when we call the second `fetch`, the result of the first one is still not resolved.

Solution: Place second `fetch` inside of `.then` of the first one. Why? Because we are sure that accountId is not an empty string in there, as the result is back from the server.

Full working code:

```js
const APPLICATION_ID = "XXXXXXXXXXXXXXXXXXXXX"; // This is your secret application key
let accountId = ""; // prepare the accountId variable
let playerStats = null; // prepare playerStats variable
let searchByNameUrl = `https://api.worldoftanks.eu/wot/account/list/?application_id=${APPLICATION_ID}&search=`;
let searchByAccountIdUrl = `https://api.worldoftanks.eu/wot/account/info/?application_id=${APPLICATION_ID}&account_id=`;

const playerName = "Komah_1"; // This is my username, you can put whatever you want here
searchByNameUrl = searchByNameUrl + playerName; // append player name to the url that we will request

fetch(searchByNameUrl)
  .then(response => response.json()) // This parses the response to JSON
  .then(result => {
    accountId = result.data[0].account_id;
    searchByAccountIdUrl += accountId; // we can write this shorter :)
    fetch(searchByAccountIdUrl)
      .then(response => response.json()) // again, convert to JSON
      .then(result => {
        playerStats = result.data;
        console.log(playerStats);
      });
  });
```

And now you can inspect `playerStats` variable in the console, and see what it contains. In case of my username, you can see below on the picture.

If you have any questions about this, feel free to send me an email.

Additional links:

- Fetch documentation - https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- Template literals - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals