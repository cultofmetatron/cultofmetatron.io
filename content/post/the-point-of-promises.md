---
Categories:
- Development
- Javascript
- Rxjs
Description: ""
Tags:
- Development
- Javascript
date: 2016-03-02T00:06:38-05:00
menu: blog
title: the point of promises
---

You've learned the basics of javascript. Hell, I dare say you've probably squeezed out a first project. I'm willing to bet that for most of you, the idea of asynchronous code still seems like magic.

True enough, javascript's event loop model of execution is strange and fantastical when you are first learning it and downright aggravating when you suddenly realize that you have a list of filenames. You need to load them all up and run another function with the contents of each respective file. 

Need to do this with two? Easy.
<!--more--> 
```language-javascript
var fs = require('fs');
fs.readFile('hello.txt', 'utf8', function(err, helloCont) {
  fs.readFile('world.txt', 'utf8', function(err, worldCont) {
  	console.log(helloCont + ' ' + worldCont);
  });
});
```

Ugly but easy enough. What if you don't know how many async calls you need to make? 

```language-javascript
var files = getFiles();
var i;
var results = [];
for(i = 0; i < files.length; i++) {
  fs.readFile(files[i], 'utf8', function(err, contents) {
  	results.push(contents);
    if (i === files.length - 1) {
     console.log(results);
    }
  }
}
```

I make no apologies. This code is ugly and boorish. More importantly, there are race conditions in this code. We have no way to guarantee the order in which the file contents will come back in. Additionally, I haven't even touched error handling. I will leave a more correct implementation as an exercise to the reader. 

This kind of code exemplifies the capacity for asynchronous node code to turn into an absolute mess. Breaking it down, this messiness arises due to a few common factors

* Asynchronous code does not enforce a standard interface for io. cb style, thunks, eventemitters etc.
* Asynchronous code does not return anything relevant. It's entirely side-effect dependent, which makes unit testing difficult if not impossible.
* Pyramids of DOOM.
* Error handling.... TOTALLY FUCKED!

To fix these, Promises povide an interface for manipulating asynchronous data through a chainable composable object. ie: a monad.

I promise it's not as scary as it sounds.

####Basics

A Promise is an object that obeys the following contraints.

1. It encapsulates the state of an asynchronous event. ie: it is either pending, resolved with a value or rejected with an error.

2. It is chainable, we can appply a transformation that returns a new instance of a promise that contains a value dependant on the eventual value of the first one.

3. It presents a *composable* unified interface. Any function that takes a promise can be agnostic to the nature of how the asynchronous value is derived. ie: it doesn't matter if the asyc event comes from fileio, setTimeout or user interaction.

4. Asynchronous functions now have a synchronous return value. This merges the conflict between callback style and functional programming.

5. It represents a one time event. For subscriptions on successive asynchronous events, you will have to fall back to eventemitters or upgrade to reactive streams.

To illustrate, let's create a basic function to return a Promise. The function timer, takes a value and a timeout and returns a Promise for that value that resolves with that value in the specified timeout.

For this example, we'll be using bluebird.js

```language-javascript
var Promise = require('bluebird');

var timer = function(value, timeout) {
  return new Promise(function(resolve, reject) {
  	setTimout(function() {
      resolve(value);
    }, timeout);
  });
};
```

The Promise constructor is passed a callback that is given two functions as arguments. resolve and reject can then be used within the callback to resolve or reject the state of the promise we are returning. Its important to note that this is happenning asynchronously. We return a promise immediately and we can worry about how the promise will resolve or reject later.


To use a Promise, we chain a function to be run when it resolves by using **.then()**. Five seconds later, "hello world" prints to the screen. 

```language-javascript
timer("hello world", 5000).then(function(value) {
  console.log(value);
});
```

Remember how I said Promises were *composable*? Calling **.then(fn)** takes a callback and returns a new Promise for the value of the first promise with the callback representing how that value is transformed.

```language-javascript
timer("hello world", 5000).then(function(value) {
  return value + ' things are great!'
})
.then(function(value) {
	console.log(value); // 'hello world things are great!'
});

```

Even better, if the callback returns a promise, the then returns a promise for the value of the eventual value of the promise returned inside the callback.

```language-javascript
timer("hello world", 5000).then(function(value) {
  return timer(" things are great!", 5000)
  .then(functon(val) {
    return value + val;
  });
})
.then(function(value) {
	console.log(value); // 'hello world things are great!'
});

```

The result is that in 10 seconds, *'hello world things are great!'* prints to the screen.

Lets rework our original file loading example earlier using Promises.

we can write a function that takes a filename and encoding and returns a Promise for the contents trivially.

```language-javascript
var readFileAsync = function(filename, encoding) {
  return new Promise(function(resolve, reject) {
  	fs.readFile(filename, encoding, function(err, contents) {
      if (err) { return reject(err) }
      else { resolve(contents); }
    });
  });
});
```

Bluebird has several helper methods for helping you compose promises. 

**Promise.all()** takes an array of promises and returns a promise for an array of their resolved values.

**Promise.props()** takes an object of promises and returns a promise for an object where the keys now refer to the resolved values.

Now we want to read both hello.txt and world.txt and print them. 

```language-javascript
  Promise.all([
    readFileAsync('hello.txt', 'utf8'),   
    readFileAsync('world.txt', 'utf8')
  ])
  .then(function(results) {
  	return results.join(' ');
  })
  .catch(function(err) {
  	console.log(err);
  })
```

This is much cleaner and easier to extend.

Every promise also has a catch method. This method will be called if an error is thrown anywhere in the chain and will skil subsequent then calls till we hit a catch. At this point we have an option for how we want to recover from the error. In this case, we simply print the error.

Writing a wrapper for every node function is however a pain. If we assume, that the function follows the pattern of callback as last argument given an err and data. Its pretty easy to write a wrapper to make converting functions easier.

The function **promisify()** takes a node callback style async function and returns a promise returning version of it.

```language-javascript
var promisify = function(fn) {
  return function(args) {
  	args = Array.prototype.slice.call(arguments);
    var self = this;
    return new Promise(function(resolve, reject) {
      args.push(function(err, data) {
        if (err) { reject(err); }
        else { resolve(data); }
      });
      fn.apply(self, args);
    });
  }
};

```

**Promise.promisify()** in bluebird does this. There is also **Promise.promisifyAll()** which takes an object and decorates the object with promise returning functions. They are appended with 'Async.' 

With this function, we can now work from a much higher level. For each file load, we can apply a catch handler in case the file does not exist. In this case, fs.readFileAsync returns a promise for the contents of the file. If there is an error, we use catch to recover by having it be a  return for an empty string.

```language-javascript
var Promise = require('bluebird');
var fs = Promise.promisifyAll(require('fs'));
var _ = require('lodash');

Promise.all(_.map(['hello.txt', 'world.txt'], function(file) {
  return fs.readFileAsync(file, 'utf8')
  .catch(function(err) {
  //we recover with a empty string if there is an error
    return ''; 
  });
}))
.then(function(results) {
  console.log(results.join(''));
})
.catch(function(err) {
  console.log(err);
})

```

Composablility, chaining and easy error propagation. 

That's the point of promises.

