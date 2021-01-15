# Disable Automatic Semicolon Insertion (ASI)

## Background Info

Javascript is an interpreted language, and there is no compilation stage that checks for syntax errors. When you deliver a script to someone in a website (for example) and it's missing a semicolon, the script would stop running because of a syntax error. Because of this, Automatic Semicolon Insertion (ASI) was created. Javascript will try to insert a semicolon to fix a syntax error instead of just crashing.

## Problem

### Inserting when not expected to

This causes issues where a semicolon would be inserted in a wrong location, causing lots of weird bugs that wouldn't have happened if ASI didn't exist.

In some cases, it inserts a semicolon in a place that one shouldn't be in. For example:

```js
// using this syntax because it makes sense in this context
function hello()
{
   return
   {
      message: "Hello World!!!!!"
   }
}
```

This code is treated as:

```js
function hello()
{
   return;
   {
      message: "Hello World!!!!!";
   }
}
```

What was intended is to return `{ message: "Hello World!!!!!" }`, but instead the function returns void, then creates a new block with a labeled `Hello World!!!!!` (this block is unreachable).

This example is adapted [from ESLint](https://eslint.org/docs/rules/semi)

### Not inserting when expected to

In another case, a semicolon wouldn't be inserted where it would be expected to. For example:

```js
var counter = {}

(function() {
   var n = 0
   counter.increment = function() {
      return ++n
   }
})()
```

This is treated the same as:

```js
var counter = {}(function() {
   var n = 0;
   counter.increment = function() {
      return ++n;
   }
})();
```

Javascript doesn't insert a semicolon after the first line. It tries to take the empty object `{}` and call it (using the function as a parameter), then calling the result again with no parameters. This doesn't work because empty object isn't callable, so it throws.

This example is taken [from ESLint](https://eslint.org/docs/rules/semi)

## Solution

At the very top of a source file (or script tag), we can do something like:

```js
"disable asi";
// not set on this, may change the contents of the string
```

Doing this disables ASI. When a parser has ASI disabled and it finds a syntax error, instead of continuing to parse the script, it stops and errors when it finds a missing semicolon.

This is similar to `"use strict";` in the way that its just a string literal at the top of a file, so its backwards compatible.

Existing languages (like Java and C++) never had ASI, mostly because unlike Javascript, they are languages that require a checking layer before deployment. Now this isn't much of a problem anymore because Javascript has a rich ecosystem of transpilers (like TypeScript and Webpack) and code linters (like ESLint). There are tonnes of ways to check code before shipping. Code editors can also check for the string at the beginning of the file and provide syntax errors when it finds a missing semicolon (like editors are doing already for other languages).
