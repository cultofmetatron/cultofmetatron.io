---
Categories:
- Development
- Javascript
Description: "How I learned to not use loops"
Tags:
- Development
- Javascript
date: 2016-05-02T00:11:43-05:00
menu: blog
title: one wierd trick to improving your javascript
---

As the complexity of the projects I tackle as an engineer have increased over the years. I've noticed that while my capacity to understand project scope have increased linearly, The problems that I've had to deal with have increased exponentially in difficulty. The human mind can only hold so much complexity at a time. To rise to the needs of being a software engineer in the 21'st century, it takes a healthy dose of humilty to recognize my weaknesses and develop a methodology to handle the complexity.
<!--more--> 
Without further ado, I introduce one of the most powerful axioms you can adopt in your own code. The extra thinking reqiuired might increase the time it takes to write the code. However, that will be more than paid back in the ability to read your own code and debug it later on.

> Use constants instead of variables and avoid mutation as much as you can.

At this point, you might be seriously questioning my sanity. As programmers, reassignment is something we do all the time. For instance, lets look at a basic loop in a factorial function.

```javascript
function factorial(n) {
  var _i;
  var acc = 1;
  for (_i = 1; _i <= n; _i++) {
    acc = acc * _i;
  }
  return acc;
}

```

This implementation relies heavily on mutation. In fact, if I were to debug this, I would have to trace through every single line of execution. However, mutation allows us to be lazy. By restricting mutation, we are forced to keep our functions as small as possible.

Realistically, we need to have code that performs mutation. We can however think about a more general case. Getting rid of mutation makes you focus on the existential description rather than the rote instructions that get you there.

Lets explore this further. A factorial n is the * operator applied on the range of numbers from 1 to n. Already, we can see that a list of numbers is involved.

Thus we need some way of creating a range(i, n) function which creates a list of numbers starting from i to n.

```javascript

function range(i, n) {
  var _i;
  var arr = [];
  for (_i = i; _i <= n; _i++) {
    arr.push(_i);
  }
  return arr;
}

```

Now we have the ability to apply multiplication across all these numbers.

```javascript

function factorial(n) {
  var acc = 1;
  var _i;
  var arr = range(1, n);
  var _l = arr.length;
  for (_i = 0; _i < _l; _i++) {
    acc = acc * arr[_l];
  }
  return acc;
}

```
This isn't much better though, we just shifted the complexity now to retrieving values from the array. We are getting closer though. There's plenty of cases where we have a sequence of values where we want to reduce them into a single scalar value. Lets push the mutation step to that helper function.

```javascript

function reduce(fn, memo, arr) {
  var acc = memo;
  var _i;
  var _l = arr.length;
  for (_i = 0; _i < _l; _i++) {
    acc = fn(acc, arr[_i], _i);
  }
  return acc;
}

```

As it turns out, we can also create range in terms of reduce.

```javascript

function range(i, n) {
  var offset = n-i;
  return reduce(function(memo, elem, index) {
    var newArray = memo.slice();
    newArray.push(index + i);
    return newArray;
  }, [], Array(offset + 1));
}

```
With these two cases taken care of, we can express factorial far more succinctly.

```javascript

function factorial(n) {
  return reduce((a, b) => a * b, 1, range(1, n));
};

```

Not only is our factorial much clearer, We now have two helper functions that we can reuse throughout the codebase.
Lets say we want to create something even crazier. Lets say we want to express the sum of the factorials from 0 to n.

```javascript
function sumofFactorials(n) {
  var factorials = reduce(function(memo, element) {
    var newArray = memo.slice();
    newArray.push(factorial(element));
    return newArray;
  }, [], range(0, n));
  return reduce((memo, element) => memo + element, 0, factorials)
}

```

Of course you might see by now that there are a lot of cases where you simply want to take an array and project a function over every value of the array and return a new array.

Array has a method called map. We can create our own using reduce.

```javascript

function map(fn, arr) {
  return reduce(function(memo, element, index) {
    var newArray = memo.slice();
    newArray.push(fn(element, index));
    return newArray;
  }, [], arr);
}

```

Now we can make our sumofFactorials even more concise.

```javascript

sumofFactorials(n) {
  return reduce(
    //es6 fat arrow syntax
    (memo, elem, idx) => { return memo + elem; },
    0,
    map(factorial, range(0,n)));
}

```

### Getting rid of loops entirely with babel

So far, we've been moving around the mutation step into smaller helper functions. In some languages like haskell, loops simply don't exist as a language construct. How is iteration accomplished? They use recursion!

Most purely functional languages support a *tail recursion*. This means that if the function returns the recursive call as the last statement, the compiler knows to unwind it into a simple loop and perform a mutation behind the scenes for extra performance.

Javascript in browsers and node.js currently don't support this. However, we can still take advantage using the [Babel.js](https://babeljs.io/) transpiler. In addition, we can now take advantage of the *const* keyword to alert us if we inadvertently reassign a variable.

Its important to note that Babel can only perform the optimization on functions declared using C-style declaration.

```javascript

function factorial(n) {
  function tailFactorial(acc, i , n) {
    if (i === n) {
      return acc * n;
    } else {
      return tailFactorial(acc * i, i++, n);
    }
  };
  return tailFactorial(1, 1, n);
}

```

Remember, the recursive call MUST be the last thing returned.

```javascript

/*
  this will not be optimized and sufficiently large
  values of n will result in a stack overflow
*/
function factorial(n) {
  function tailFactorial(acc, i , n) {
    if (i === n) {
      return acc * n;
    } else {
      return tailFactorial(acc * i, i++, n);
    }
  };
  /*that extra (* 1) forces the interpreter
    to retain the stack frame. */
  return tailFactorial(1, 1, n) * 1;
}

```


Earlier, we pushed all the looping logic to the reduce function. Lets replace it with a tail recursive version.

```javascript
function reduce(fn, memo, arr) {
  function reducer(fn, memo, arr, index) {
    return (arr.length === 0) ?
      memo : reducer(fn,
                     fn(memo, arr[0], index ),
                     arr.slice(1),
                     index + 1);
  }
  return reducer(fn, memo, arr, 0); // BOOSH
}

```

And now we've effectively eliminated all mutation in our code.

### Less Moving parts => Better code

Already you can see that by observing the rule of avoiding mutation, we have less moving parts throughout the code. If you run into a bug, its as simple as exploring the values through the stack trace to deterime where a potential bug might reside. My own experience shows that 10% of your time is spent writing the first version of the code. The rest of the time is spent debugging and refactoring this code.

As I've demonstrated, simply following the rule of avoiding mutation and variable reassignment. You as a programmer are forced to write better abstractions that reduce the signal to noise ratio of your code.

Unfortunately, the native javascript data types do not lend themselves well to being used in an immutable style. This leads to the explicit copying of structures as we pass through the stack frame. Why copy these structures in the first place? Is it worth the extra hit to memory? I would argue that it is in most cases because we can by default avoid a multitude of errors.

In practice it helps to use a library like [Immutable.js](https://facebook.github.io/immutable-js/).  It guarantees that all modifications return new copies of the structure. If passed into two diffrent functions, you can be certain the operations on the structure in one function will not affect the outcome of the other function.
