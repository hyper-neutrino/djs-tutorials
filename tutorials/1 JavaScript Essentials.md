# JavaScript Essentials

This tutorial will show you how to make a Discord bot in JavaScript using the discord.js library. First, let's go over how to set up a JavaScript environment.

## Setup

### Downloading Node.JS

You will need to install node on your computer first (node.js lets you run JavaScript locally in the terminal, as opposed to on a browser, and is necessary for things like making a web server or for our use case, making a Discord bot). You can download it on the official website at https://nodejs.org/en/download/ or follow the tutorial at https://nodejs.dev/learn/how-to-install-nodejs.

Next, running `node` should bring up a REPL. Try typing `1 + 2` and hitting <kbd>Enter</kbd>. You should see `3` appear below Now, you can hit <kbd>Ctrl</kbd> + <kbd>D</kbd> to close the REPL (or enter `process.exit(0)` for the JavaScript way to exit).

You should have also gotten a program called `npm` (Node Package Manager). You will be using this to install libraries. Try `npm --version` to confirm that this works.

### Setting up an environment

I personally use VSCode for developing in JavaScript. You should use whatever you are most comfortable with, but if you haven't used any before, my recommendations would be VSCode or Atom.

Set up a workspace the same way you usually would, preferably with a folder dedicated to your project. This can be a git repository or just a normal folder if you don't plan on using git.

To begin, open your console in this folder. If you are on Windows, you can right click in File Explorer inside your folder and click "Open Terminal Here" (you may need to press shift + right click if this option does not show up). Other operating systems likely have a similar option.

Let's start by creating a file and naming it `hello.js`. In it, put the text `console.log("Hello, World!");` and save it.

Now, go back into your terminal and running `node hello.js`. You should see `Hello, World!` appear.

## Asynchronous programming

Now that JavaScript is working, let's take a brief detour to talk about asynchronous code flow. Most programming is synchronous, meaning each operation completes before the next one starts and the code runs line-by-line. For example, if I write a program that takes user input (waiting until it receives it), processes it, and then outputs, that would be synchronous.

On the other hand, an asynchronous program would not wait ("block") for certain actions. For example, I could write the program to instead process and output when a user inputs, but not wait for it. This has several advantages. Note that this is not to be confused with multi-threading or parallel computation where multiple operations are done at once (this is how GPUs are able to render graphics so quickly, because most graphics computations do not depend on each other and thus the GPU can perform thousands of simple calculations at the same time rather than doing them one-by-one). Asynchronous code (on its own) does not run multiple operations at once, it just doesn't need to block for certain actions and can run something else while a synchronous program would otherwise be waiting.

Asynchronous programming is essential to something like a Discord bot. Let's say you have a command, `!wait`, that waits for 3 seconds and then sends a reply. If this blocked, then sending `!wait` many times would quickly stop your bot from working by clogging it up with wait operations. However, if you process them asynchronously, then `!wait` would tell the computer "when 3 seconds are up, send this". The computer now waits 3 seconds internally but immediately is ready for another operation. So, if the user sends `!wait` again after 1 second, the computer does the same thing and then sends two messages, one 3 seconds later and one 4 seconds later.

This is a massive oversimplification of asynchronous programming but the core concept is that when the Discord library receives an action, it sends it to your code to handle but doesn't wait for you to finish. This means that if your code takes a long time, another incoming message might be able to be handled first (note that this doesn't just prevent lag from being an issue; code efficiency is still crucial because a computer only has finite resources).

Asynchronous programming has already existed in JavaScript for a long time in the form of `Promise`s. In fact, JavaScript doesn't really have many blocking operations and prefers to handle things asynchronously (which makes sense because it was built for websites and you typically don't want user actions to freeze everything else as that is bad UX). A `Promise` is an object that will resolve to its result eventually. Note that it can also error, and despite being called a promise, it can actually just never resolve (either successfully or erroneously); however, this usually doesn't happen and there is not much of a reason to attempt to handle this case in most situations.

You can call `.then` on a promise with a function, which is run when the promise resolves. You can also call `.catch` which is run when the promise dies. There is of course no way to check if a promise doesn't resolve, because if it hasn't resolved, we cannot be certain that it never will. Note that a promise can also resolve with no result - this just means that it is finished and the next step can happen, which can be useful for if the promise was performing some setup that is necessary before continuing.

In recent version of JavaScript, there is a better way of handling async using the `async` and `await` keywords. Declaring a function as `async` means that the function is allowed to be run asynchronously and its result is automatically a promise. `await` is a keyword that you can use before a `Promise` which "blocks" until it resolves and then fetches the result. `await` can only be used within `async` functions and so since the entire function is being run asynchronously, it doesn't really block, it just abstracts away the `.then`.

For example, if you have three async operations in a row, you would normally need to nest `.then` multiple times which might look something like this:

```js
f().then(() => {
    g().then(() => {
        h().then(() => {
            // something
        });
    });
});
```

This is pretty ugly. However, with `async`/`await`, we can instead do the following:

```js
async () => {
    await f();
    await g();
    await h();
};
```

This ensures that `f`, `g`, and `h` are resolved in order and doesn't require an ugly chain of nested `.then`s. Don't worry about the other parts of the syntax, I'll get to that later. The takeaway from this is that `async` functions essentially allow you to write the contents of the function as though you were programming synchronously, but then converts the entire thing to asynchronous execution.

## Basic JavaScript Fundamentals

JavaScript is a dynamically typed, multi-paradigm language with first-class functions. Dynamic typing means that variables don't have a fixed type, so you can set a variable to an integer and later a string. Multi-paradigm means that it supports a variety of paradigms such as event-driven, functional, or imperative. You don't really need to worry about what these mean, it basically means it's flexible for multiple coding styles and needs. JS has first-class functions which just means that functions are also objects so you can treat them the same way you treat any other object - you can store them in variables, pass them as arguments to other functions, etc. This is in opposition to languages like Java, where functions are a totally separate entity from objects.

JavaScript uses C-style braces and optionally uses semicolons. By optional, I mean that it will automatically use semicolons where it can infer where they should go. However, I strongly recommend putting in semicolons yourself, because sometimes it doesn't insert them in the places you might think they would go, and this can cause some very weird bugs that may be hard to find (if you have a code formatter, usually the way it formats the code will make these issues apparent, but it is still best not to rely on that).

### Value Types

In JavaScript, you will mostly be working with the following types of objects:

-   integers: whole numbers; unlike some languages, these have limited precision so beyond a certain range integers stop working correctly
-   floats: floating point numbers; like most languages, these also have limited precision
-   strings: basically just text
-   lists: an ordered collection of other objects
-   others: you will be using a variety of other objects; you can also make your own, which we will go over later

Note that there are also two special values, `null` and `undefined`. They both represent the absence of a value but they are slightly different; `null` is similar to "this value is defined to be absent" and `undefined` is "the definition of this value is missing".

### Operators

JavaScript has most of the same operations as you'd expect from any other language and should be intuitive enough.

One set of operations that is quite uncommon that JavaScript is the following - `x ?? y` will be equal to `x` if `x` isn't `null` or `undefined`, and `y` if it is. Note that most languages have something similar in the form of a logical OR operation (JavaScript has this as well) - `x || y` will return `x` if `x` is a truthy value and `y` otherwise. However, these two operations differ when `x` is defined but falsy; for example, `false || y` is `y` but `false ?? y` is `false`. Likewise, `0 || y` is `y` and `0 ?? y` is `0`.

A final syntactical component I will talk about is `?.`. In JavaScript, you access fields using `.` - for example, if you have a `TextChannel` object, you can do `channel.topic` to access its topic ([documentation reference](https://discord.js.org/#/docs/discord.js/stable/class/TextChannel?scrollTo=topic)). If the object you are accessing is `undefined`, this will throw an error. However, if you do `a?.b`, instead of erroring, it will return `undefined` if `a` is `undefined`. In combination with `??`, you can do something like `a?.b ?? c` to easily reduce what could have ended up as a nested `if`-statement into a single clean expression.

The following is a full list of operations which should be fairly intuitive:

|   Operator   |                                                         Description                                                         |
| :----------: | :-------------------------------------------------------------------------------------------------------------------------: |
|     `+`      |                         Addition (can concatenate strings including adding them with other objects)                         |
|     `-`      |                                                         Subtraction                                                         |
|     `*`      |                            Multiplication (cannot repeat strings, use `"...".repeat(x)` instead)                            |
|     `**`     |                                              Exponentiation (added in ES 2016)                                              |
|     `/`      |                                                          Division                                                           |
|     `%`      |                                                     Modulus (Remainder)                                                     |
|     `++`     | Increment (`++x` and `x++` both increase `x` by one; the former returns the new value and the latter returns the old value) |
|     `--`     |                                      Decrement (same as above but decrease `x` by one)                                      |
|     `==`     |               Equal To (note: this will convert between types sometimes unexpectedly; `0 == ""` for example)                |
|    `===`     |                Equal Value and Equal Type (it is not the case that `0 === ""`, so be careful which you use)                 |
|     `!=`     |                                               Not Equal To (opposite of `==`)                                               |
|    `!==`     |                                     Not Equal Value and Equal Type (opposite of `===`)                                      |
|     `>`      |                                                        Greater Than                                                         |
|     `<`      |                                                          Less Than                                                          |
|     `>=`     |                                                  Greater Than or Equal To                                                   |
|     `<=`     |                                                    Less Than or Equal To                                                    |
|    `? :`     |                  Ternary Operator; `a ? b : c` evaluates to `b` if `a` is a truthy value and `c` otherwise                  |
|     `&&`     |                               Logical AND; `a && b` is `a` if `a` is falsy and `b` otherwise                                |
|   `\|\| `    |                                 Logical OR; `a \|\| b`is`a`if`a`is truthy and`b` otherwise                                  |
|     `!`      |                          Logical NOT; `!a` is `false` if `a` is truthy and `true` if `a` is falsy                           |
|   `typeof`   |                                          `typeof a` returns `a`'s type as a string                                          |
| `instanceof` |                                  `a instanceof B` returns whether or not `a`'s type is `B`                                  |
|     `&`      |                                                         Bitwise AND                                                         |
|     `\|`     |                                                         Bitwise OR                                                          |
|     `~`      |                                                         Bitwise NOT                                                         |
|     `^`      |                                                         Bitwise XOR                                                         |
|     `<<`     |                                                         Left Shift                                                          |
|     `>>`     |                                                         Right Shift                                                         |
|    `>>>`     |                                                    Unsigned Right Shift                                                     |
|     `[]`     |                                               Index Access (`object[index]`)                                                |
|     `.`      |                                                Field Access (`object.field`)                                                |

The last few are _bitwise_ operations - they work between two integers and operate on their binary representations. Essentially, you convert the numbers into binary and operate per pair of bits, so `10 & 12 == 8` because `10` is `1010` and `12` is `1100`, and if you take each pair of bits, you get `(1&1) (0&1) (1&0) (0&0)` which is `1000`. For more information, check out the W3Schools page on [JavaScript Operators](https://www.w3schools.com/js/js_bitwise.asp). You will likely not need to use these.

Also, `[]` and `.` are not really operators. `x[y]` will return the `y`th element of a list `x` (note that `x[0]` is the first element), or if `y` is a string, is equivalent to `x.(the value of y)`. For example, if `y == "abc"`, then `x[y] === x.abc`.

### Defining Variables

JavaScript variables technically do not need to be declared, unlike in some languages like C. You can just do `a = 3`. However, you should not do that without first declaring the variable. There are three ways to declare variables in JavaScript - `var`, `let`, and `const`. `const` variables cannot be redefined; they can still be mutated, but you cannot re-assign them.

The difference between `var` and `let` comes down to scope, i.e. where the variable is defined in. Variables defined with `var` are bound to the function's scope. Consider the following example:

```js
function f() {
    if (true) {
        var a = 3;
    }
    console.log(a);
}
```

When you call `f()`, it will output `3`. On the other hand, `let` will bind the variable's scope to the immedate enclosing block (`{ ... }`), so in the example above, if we replaced `var` with `let`, we would get an error, since `a` stops being defined outside of the `if { ... }` block.

Which one you use depends on your use case, but in general, I would recommend always using `let`. If you need to do something like what I showed above, simply do the following:

```js
function f() {
    let a;
    if (true) {
        a = 3;
    }
    console.log(a);
}
```

Variables are always accessible within a _narrower_ scope, and this way, you can control exactly in what scope you want your variable to be defined. In fact, `let` was added because there were so many bugs caused by people not understanding how `var` worked that they made a whole new keyword to let (pun intended) you control your variables better.

There are some more nuances; you can read about them (and this scoping behavior) in [this excellent Stack Overflow answer](https://stackoverflow.com/a/11444416/8200485).

Also, `const` is the same as `let` except you can't redefine the variable.

Once a variable has been declared with `var` or `let`, you can simply do `name = value` to re-assign it. You can also do `name += value` which is essentially `name = name + value`, and likewise with most other operators.

### Functions

Functions are a crucial part of JavaScript. They are essentially blocks of code that you can then run from other places. You can treat a function as an input-output machine - once you've defined the machine, you can provide it certain inputs and obtain an output, and the function may also have some side effects like outputting to the console.

There are two ways to define functions: the standard notation using the `function` keyword, and arrow functions which are more new and not supported by all browsers (however, since you are using node.js, if it works on your machine, that is good enough).

```js
function function_name(x, y, z) {
    // do something
    return 0;
}
```

This defines a basic function that takes three values and returns `0`. There are a few things to unpack here. `x`, `y`, and `z` can be any objects, or can even be undefined. They are treated as function-scoped variables (and are not `const`), meaning you can re-assign them inside the function body. This will _not_ re-assign the original variable; for example, if I define `function f(x) { x = 3; return 0; }`, then `let x = 2; f(x);` will not alter the value of `x` outside of the function.

`return` does two things. Firstly, it defines the result of calling the function, so you can do `let k = f(2)` and you will get `k == 0`. Secondly, it immediately exits the function.

The second way to define functions is with arrow notation:

```js
(x, y, z) => {
    // do something
    return 0;
};
```

Note that there is no way to define arrow functions with a name. You can just do `let function_name = (...) => { ... }`. You can actually define anonymous functions using the keyword as well:

```js
function(x, y, z) {
    // do something
    return 0;
}
```

This creates a function but doesn't name it. You can assign it a name using `let name = function(...) { ... }` - I'll explain how this is different in a bit. You also don't have to name them - recall that I said that JavaScript has first-class functions. This means that you can create a function to pass into another function; for example, if I have a function `f` that accepts another function, I can do `f(function() {})` or `f(() => {})`.

Finally, there is one more thing to know about arrow functions - you do not have to define them with a block (`{ ... }`):

```js
let add = (x, y) => x + y;
```

This is equivalent to `let add = (x, y) => { return x + y; }`. Essentially, if you only have one expression, you do not need `{}`, and it will also return the result of the expression.

### Asynchronous Functions

The syntax is as follows:

```js
async function f(x, y) {
    return x + y;
}
```

In this case, `f(3, 4)` will not return `7`; rather, it will return a `Promise`. If you do `f(3, 4).then(console.log)`, you will see `7` show up in the output. Also, if you do `await f(3, 4)`, then you will get `7`.

```js
let f = async (x, y) => x + y;
```

This is the same. You can then use `await` within `async` functions including single-expression arrow functions:

```js
let g = async (x) => await f(x, x);
```

(The above is actually equivalent to `let g = x => f(x, x)` because you don't need to explicitly make the function async and then await the result, you can just return the promise itself, since you can `await` non-`async` functions if they return `Promise`s).

### Conditionals

There are a few ways to branch, or run different code based on conditions. The simplest is the `if` statement:

```js
if (condition) {
    statement;
}
```

`statement` will be run if and only if `condition` is true. A natural extension of that is the `else` block:

```js
if (condition) {
    a;
} else {
    b;
}
```

If `condition` is true, then `a` is run, and otherwise, `b` is run. Note that the `{}` are not necessary if you only have one statement. You can also write `else if` to run a block if the previous statement was false but the next one is true:

```js
if (x) {
    a;
} else if (y) {
    b;
} else {
    c;
}
```

Here, if `x` is true, `a` is run, and then the remainder is completely ignored, even if `y` is true, because it is an `else if`, meaning it only runs if `x` _isn't_ true.

If `x` is false, and `y` is true, then `b` is run. Otherwise, if `x` and `y` are both false, `c` is run. You can chain as many `else if` blocks as you want but each chain must start with an `if` and there can only be one `else`.

Another way to do conditionals is the ternary operator which I skimmed over earlier. `x ? y : z` will return `y` if `x` is true and otherwise `z`, but not only does it affect the return value, it also affects the execution. `y` and `z` will not both be evaluated, meaning if one of them would error, the error will not happen if it gets ignored by the ternary. Essentially, it evaluates `x`, and then if it's true, evaluates and resolves to `y`, and otherwise, evaluates and resolves to `z`.

This is actually the same for `||` and `&&` - if `x` is truthy then `x || y` will not evaluate `y`, and if `x` is falsy then `x && y` will not evaluate `y`.

The last common way to do conditionals is the `switch-case` construct:

```js
switch (value) {
    case 1:
        a;
        break;
    case 2:
        b;
    case 3:
        c;
        break;
    default:
        d;
        break;
}
```

This is a nicer syntax for if you are just comparing an object for equality against multiple possible cases, and is cleaner than a line of `if-else if-else` chains. There is one other caveat - the `break` statement exits the `switch` block. If you don't `break` within a case, it will actually go to the next case. So in this example, if `value == 1`, then only `a` is run. If `value == 2` however, both `b` and `c` will be run, because there was no `break` at the end of `case 2`. The `default` statement is optional and is run if all of the other cases fail.

### Loops

JavaScript has four types of loops - for-each loops, C-style for loops, while loops, and do-while loops. I'll go through them in that order.

```js
for (let x of a) {
    console.log(x);
}
```

This will loop through `a` and run the block inside once per value, assigning the value to `x`. You can use `var`, `let`, or `const` here.

If instead of `of`, you use `in`, it will instead loop through the keys. This means that if you have a custom object, it will loop through the field names as strings, and if you loop through an array, it will loop through the indexes.

```js
for (pre; condition; post) {
    ...
}
```

This will first run `pre`, and then start a loop. Each time, if `condition` is true, it will run the loop once. At the end of that, it will run `post`. Then, it returns to the top and checks again. Thus, `pre` is only run once, and `condition` is run once before each loop and before the loop stops, and `post` is run once at the end of each loop.

If `pre` or `post` are blank, nothing happens, and if `condition` is blank, it is implicitly treated as true. Thus, `for(;;) {}` is an empty infinite loop.

```js
while (condition) {
    ...
}
```

This one is the easiest to understand. Each time, it checks if `condition` is true. If so, it runs the block and then returns to the top and checks again. If not, it exits.

```js
do {
    ...
} while (condition);
```

This one is almost the same as the while loop with one caveat - it checks the condition at the end. Thus, if the condition starts out false, the while loop will run 0 times, whereas the do-while loop will run once.

In all loops, the statement `break` will exit the loop and go past the end, and `continue` will go back to the top (including running the condition check again, or for for-each loops, it will go to the next item). Note that it does not skip `post` in three-part for loops. For example:

```js
for (let x = 0; x < 10; x++) {
    if (x == 5) continue;
    console.log(x);
}
```

This will output all numbers from 0 to 9 except 5. If you replace the `continue` with `break`, it will output 0 to 4, and then when it reaches 5, it will instead just exit the loop and stop the future iterations as well.

Finally, except for the do-while loop, you can remove the `{}` if you only have one statement.

### Destructuring Assignment

Lastly, let's look at destructuring assignment. Firstly, in JavaScript, objects are created using the syntax `{ name: value, name: value, ... }`. For example, we could do `const x = { a: 3, b: 4 }`. The value can be anything including other objects, and the name should be just a name or it can also be a string. To access these values, we can do `x["a"]` or `x.a`. The property names do not need to be valid variable names; for example, I could do `const x = { "!": 1 }` and then do `x["!"]`; however, then I would not be able to do `x.!`.

Another quick note: `{ a }` is valid if `a` is defined here; it is equivalent to `{ a: a }`.

Destructuring assignment essentially attempts to match the structure of the object to the structure of the left side of the assignment. For example, let's say we have an object `let x = { a: 1, b: [ 3, 4 ] }`. Then, if I do `let { a } = x`, this will assign `a` equal to `1` and ignore `b`. Basically, it matches the structure so since on the left side I want an object with a property `a`, it looks on the right side and gets the property `a` of the object `x`. If I do `let { b } = x`, then `b` now equals `[3, 4]`.

This also works on lists: if I `let x = [1, 2, 3]` and then `let [a, b] = x`, now `a == 1` and `b == 2`.

Destructuring assignment is extremely powerful especially because you can actually nest it. Consider the following example: `let x = [1, { a: 2, b: 3 }]`. Now, if I do `let [q, { a, b }] = x`, we get `q == 1`, `a == 2`, and `b == 3`.

What if you want to do `let a = x.b`; that is, you don't want to keep the same name? That is quite simple - just do `let { b: a } = x`. Basically, this will find the property `b` of `x` and then assign it to `a` instead of `b`.

---

Next time, we will look at the basics of **discord.js** and how to write a basic bot.
