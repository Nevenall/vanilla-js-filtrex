Filtrex
=======
[![Build Status](https://travis-ci.com/m93a/filtrex.svg?branch=master)](https://travis-ci.com/m93a/filtrex)

A simple, safe, JavaScript expression engine, allowing end-users to enter arbitrary expressions without p0wning you.

```python
category == "meal" and (calories * weight > 2000.0 or subcategory in ("cake", "pie"))
```

Why?
----

There are many cases where you want a user to be able enter an arbitrary expression through a user interface. e.g.

*   Plot a chart ([example](http://rawgit.com/joewalnes/filtrex/master/example/plot.html))
*   Filter/searching across items using multiple fields ([example](http://rawgit.com/joewalnes/filtrex/master/example/highlight.html))
*   Colorize items based on values ([example](http://rawgit.com/joewalnes/filtrex/master/example/colorize.html))
*   Implement a browser based spreadsheet

Sure, you could do that with JavaScript and `eval()`, but I'm sure I don't have to tell you how stupid that would be.

Filtrex defines a really simple expression language that should be familiar to anyone who's ever used a spreadsheet and compile it into a JavaScript function at runtime.

Features
--------

*   **Simple!** End user expression language looks like this `transactions <= 5 and abs(profit) > 20.5`
*   **Fast!** Expressions get compiled into JavaScript functions, offering the same performance as if it had been hand coded. e.g. `function(item) { return item.transactions <=5 && Math.abs(item.profit) > 20.5; }`
*   **Safe!** You as the developer have control of which data can be accessed and the functions that can be called. Expressions cannot escape the sandbox.
*   **Pluggable!** Add your own data and functions.
*   **Predictable!** Because users can't define loops or recursive functions, you know you won't be left hanging.

This Fork
------

This is intended to provide a vanilla js package that uses es5 modules. 

10 second tutorial
------------------

```javascript
// Input from user (e.g. search filter)
var expression = 'transactions <= 5 and abs(profit) > 20.5';

// Compile expression to executable function
var myfilter = compileExpression(expression);

// Execute function
myfilter({transactions: 3, profit:-40.5}); // returns 1
myfilter({transactions: 3, profit:-14.5}); // returns 0
```

Under the hood, the above expression gets compiled to a clean and fast JavaScript function, looking something like this:

```javascript
// Resulting function
function(item) {
  return item.transactions <= 5 && Math.abs(item.profit) > 20.5;
}
```

Expressions
-----------

There are only 3 types: numbers, strings and arrays of these. Numbers may be floating point or integers. Boolean logic is applied on the truthy value of values (e.g. any non-zero number is true, any non-empty string is true, otherwise false).

Okay, I lied to you, there are also objects whose properties can be accessed by the `of` operator. And there's undefined. But everything else is just numbers, strings and arrays!

Values | Description
--- | ---
43, -1.234 | Numbers
"hello" | String
" \\" \\\\ " | Escaping of double-quotes and blackslash in string
foo, a.b.c, 'foo-bar' | External data variable defined by application (may be numbers or strings)

Numeric arithmetic | Description
--- | ---
x + y | Add
x - y | Subtract
x * y | Multiply
x / y | Divide
x % y | Modulo
x ^ y | Power

Comparisons | Description
--- | ---
x == y | Equals
x != y | Does not equal
x < y | Less than
x <= y | Less than or equal to
x > y | Greater than
x >= y | Greater than or equal to
x ~= y | Regular expression match
x in (a, b, c) | Equivalent to (x == a or x == b or x == c)
x not in (a, b, c) | Equivalent to (x != a and x != b and x != c)

Boolean logic | Description
--- | ---
x or y | Boolean or
x and y | Boolean and
not x | Boolean not
x ? y : z | If boolean x, value y, else z
( x ) | Explicity operator precedence

Objects and arrays | Description
--- | ---
(a, b, c) | Array
a in b | Array a is a subset of array b
x of y | Property x of object y

Built-in functions | Description
--- | ---
abs(x) | Absolute value
ceil(x) | Round floating point up
floor(x) | Round floating point down
log(x) | Natural logarithm
max(a, b, c...) | Max value (variable length of args)
min(a, b, c...) | Min value (variable length of args)
random() | Random floating point from 0.0 to 1.0
round(x) | Round floating point
sqrt(x) | Square root

Operator precedence follows that of any sane language.

Adding custom functions
-----------------------

When integrating in to your application, you can add your own custom functions.

```javascript
// Custom function: Return string length.
function strlen(s) {
  return s.length;
}

let options = {
  extraFunctions: { strlen }
};

// Compile expression to executable function
let myfilter = compileExpression('strlen(firstname) > 5', options);

myfilter({firstname:'Joe'});    // returns 0
myfilter({firstname:'Joseph'}); // returns 1
```

Custom property function
------------------------

If you want to do some more magic with your filtrex, you can supply a custom function that will resolve the identifiers used in expressions and assign them a value yourself. This is called a _property function_ and has the following signature:

```typescript
function propFunction(
  propertyName: string, // name of the property being accessed
  get: (name: string) => obj[name], // safe getter that retrieves the property from obj
  obj: any // the object passed to compiled expression
)
```

For example, this can be useful when you're filtering based on whether a string contains some words or not:

```javascript
function containsWord(string, word) {
  // your optimized code
}

let options = {
  customProp: (word, _, string) => containsWord(string, word)
};

let myfilter = compileExpression('Bob and Alice or Cecil', options);

myfilter("Bob is boring"); // returns 0
myfilter("Bob met Alice"); // returns 1
myfilter("Cecil is cool"); // returns 1
```

**Safety note:** The `get` function returns `undefined` for properties that are defined on the object's prototype, not on the object itself. This is important, because otherwise the user could access things like `toString.constructor` and maybe do some nasty things with it. Bear this in mind if you decide not to use `get` and access the properties yourself.

FAQ
---

**Why the name?**

Because it was originally built for FILTeR EXpressions.

**What's Jison?**

[Jison](http://zaach.github.io/jison/) is bundled with Filtrex – it's a JavaScript parser generator that does the underlying hard work of understanding the expression. It's based on Flex and Bison.

**License?**

[MIT](https://github.com/joewalnes/filtrex/raw/master/LICENSE)

**Unit tests?**

Here: [Source](https://github.com/joewalnes/filtrex/blob/master/test/filtrex-test.html), [Results](https://rawgit.com/joewalnes/filtrex/master/test/filtrex-test.html)

**What happens if the expression is malformed?**

Calling `compileExpression()` with a malformed expression will throw an exception. You can catch that and display feedback to the user. A good UI pattern is to attempt to compile on each keystroke and continuously indicate whether the expression is valid.


Contributors
------------

* [@joewalnes](https://github.com/joewalnes) Joe Walnes – the author of this repository
* [@m93a](https://github.com/m93a) Michal Grňo – maintainer of the NPM package and the current main developer
* [@msantos](https://github.com/msantos) Michael Santos – quoted symbols, regex matches and numerous fixes
* [@bradparks](https://github.com/bradparks) Brad Parks – extensible prop function
* [@arendjr](https://github.com/arendjr) Arend van Beelen jr. – quote escaping in string literals
* [@alexgorbatchev](https://github.com/alexgorbatchev) Alex Gorbatchev – the original maintainer of the NPM package


Like this? Want other thingies?
-------------------------------

*   [meta-utils](https://github.com/meta-utils) – Various useful utilities for TypeScript and JavaScript development
*   [websocketd](https://github.com/joewalnes/websocketd) – Turn any program that uses STDIN/STDOUT into a WebSocket server. Like inetd, but for WebSockets.
*   [ReconnectingWebSocket](https://github.com/joewalnes/reconnecting-websocket) – Simplest way to add some robustness to your WebSocket connections.
*   [Smoothie Charts](http://smoothiecharts.org/) – JavaScript charts for streaming data.
*   Visit [The Igloo Lab](http://theigloolab.com/) to see and subscribe to other thingies I make.
