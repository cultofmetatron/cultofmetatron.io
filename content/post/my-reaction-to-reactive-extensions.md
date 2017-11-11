---
Categories:
- Development
- Javascript
Description: "rxjs is cool"
Tags:
- Development
- Javascript
date: 2016-04-02T00:09:41-05:00
menu: blog
title: my reaction to reactive extensions
---

Every javascript programmer is on their own hero's journey to mastery.

It begins with the basic language. Before you know it, You run into the initially paradoxic behavior of asynchronous programming. Trying to chain diffrent asynchronous functions results in nested callbacks. We all get to a point we've written a program and found ourselves entombed in our own Pyramid of Doom.

Escaping this nested hell of callbacks results in learning about helper libraries like async. We start using event emitters which allow us to pass data between diffrent parts of the system in a decoupled fashion.
<!--more--> 
However, There is a problem with callbacks and event emitters. In functional programming, functions perform no side effects and systems are statefree. An asycnhronous function invariably relies on the existence of state. This is anti-functional. This makes unit testing and breaking programs into smaller units, which are easier to reason about, much harder.

From functional programming, The Promise abstraction brought a composable interface for working with asynchronous functions. Now, an asyhcronous function can return an object immediatly that provides an interface for composing values on data that did not yet exist. We also get ways of propagating errors if they occurred. 
```javascript
function() {
  return doSomething()
  	.then(function(something) {
    	tryThis(something)
    })
    .catch(function(err) {
    	handleError(err)
    });
});

```

Unfortunatly, Promises only cover one shot asynchronous values. For sequences of data, most engineers fallback to event emitters.

Meanwhile, in academia, there's been research on an interesting new paradigm called "functional reactive programming." It was created as a way to resolve the seemingly dialectically opposed views of imperative and functional programming.

Rx.js is a library for functional reactive programming in javascript backed by Microsoft.  The more I use it, the more I realize that this is the way we should all be coding.

Reactive programming is the idea that the the result of the computation in some way should take the initiative of passing its values to all dependencies on that value. In Rx.js, data is organized around the **Observable**.

The Observable is a box like object that encapulates a stream of data for an associated event. For all intents and purposes, you can imagine its an array where values get pushed in over time. You can chain callbacks into it to return new Observable that reflect a change to the Observable. It allows you to model the flow of your program the way you would model array transforms in lodash or underscore.

More importantly, like Promises, Observables can be composed. The browser has a 'keydown' event and a 'keyup' event. What if I wanted to register a key push and release event?

Here's a naive first try you might recognize.

```javascript
$(document).on('keydown', function(ev) {     	  
  $(document).on('keyup', function() {
    console.log('keyup')
    doSomething();
    $(document).off('keyup');
  });
});
```

Of course its incredibly naive. What if there are multiple handlers on document's 'keyup'? the complexity of trying to coordinate reactions to chains of multiple events only compounds.

Observables make this a piece of cake.

```javascript
var Rx = require('rx');
var keyupStream = Rx.Observable.fromEvent(document, 'keyup');

var keydownStream = Rx.Observable.fromEvent(document, 'keydown');

var keyPressStream = keydownStream.sample(keyupStream)
  .subscribe(doSomething, onError);
```

By using the Rx convenience method, we can get an Observable for the event in which a user pushes down on a key and releases. When we **subscribe()** to it, events coming down percolate to the callback we provide. the second argument is passed any errors that come up.

We can do so much more though. What if we only want a keypress for the enter key? and we want to modify the data being passed to a string that we can look up in a table somewhere else in our system?

```javascript
var enterKey = 13 //the charcode.
keydownStream.sample(keyupStream)
	.filter(function(ev) {
    	return ev.keyCode === enterKey //c
    })
    .map(function(ev) {
    	return ev.keyCode
    })
    .subscribe(doSomething, onError);
```

It doesn't take much to see how this abstraction can expand to cover very complex interactions. 

Say you want a stream for the velocity vector of the mousemove. Not a very easy thing to do generally, you have to sample the last two mouse interactions, grab the positions and compute the dy and dx.


Using a few more methods on [Observable.](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md) This is pretty simple.

```javascript
var mouseMove = Rx.Observable.fromEvent(document, 'mousemove');

mouseMove
  .map(function(ev) {
    return {
      x: ev.clientX,
      y: ev.clientY
    };
  })
  .pairwise(2)
  .map(function(polls) {
    var pos1 = polls[0];
    var pos2 = polls[1];
    return {
      x: pos2.x - pos1.x,
      y: pos2.y - pos1.y
    }
  })
  .subscribe(function(ev) {
    console.log(ev.x, ev.y)
  }, function(err) {
    console.log(err);
  });
```

Now everytime I move a cursor, a pairwise velocity vector prints out. I can see interesting applications applied to interactions like [Amazon's dropdown menu.](http://bjk5.com/post/44698559168/breaking-down-amazons-mega-dropdown) Everyone is moving to react.js. Why not have react views be connected to data flowing in over composable event streams?

There's alot of rethinking about how we have to organize and write our apps to use Rx but the more I use it, the more I'm convinced that this is the future.


