---
layout: post
title:  "JavaScript's type system"
comments: true
date:   2016-04-21 17:59:12 +1000
categories: code javascript down-the-rabbit-hole
---

I often hear JavaScript criticised for its type systems and concatenation process. While I'm not here to argue that throwing exceptions at the first sign of trouble is the way to go, I do believe that JavaScript isn't a silly, nonsensical language designed for the mad hatter and his tea party. It _can_ make some sense if you let it.

#Everything in JavaScript is an Object

Literally everything in JavaScript is an object, there aren't things that are only "_values_".

These objects aren't empty or filled just with the value either, they come with a few things, like prototypes and inherited properties (depending on the type).

Types that aren't extensible (try running [`Object.isExtensible`][1] on the variable to see) are commonly mistaken for value types. Extensible types can be assigned properties.

Some of the primitive types (`string`, `number` and `boolean`) can be created without constructor style creation:

    var str = ""; // string
    var num = 42; // number
    var bool = false; // boolean

However, the constructor wrapper creation for these variables creates an extensible type:

    var str = new String(""); // String
    var num = new Number(42); // Number
    var bool = new Boolean(false); // Boolean

Which is why when you try to examine `str`, `num` and `bool` in a REPL environment, you get an object returned.

#What really happens when you concatenate:

If the types aren't a primitive type, they call an function called `ToPrimitive`, which is attached to the `Symbol` prototype chain. `ToPrimitive` calls two publicly accessible functions called `valueOf` and `toString`.

`valueOf` just returns the value, but `toString` is a little more complicated.

There's some types that inherit the object type, and they're designed to return `[object Type]` in `toString` if the type doesn't have their own implementation.

[Quite a few many types][2] inherit the object type:

 - `Object`
 - `Function`
 - `Array`
 - `String` (wrapper)
 - `Boolean` (wrapper)
 - `Number` (wrapper)
 - `Date`
 - `RegExp`
 - `Error`
 - `EvalError`
 - `RangeError`
 - `ReferenceError`
 - `SyntaxError`
 - `TypeError`
 - `URIError`

Most of them have their own `toString` implementation, as a decent type should, however you can see the `[object Type]` effect by running `Object.prototype.toString.call()` over a variable of that type.

    let now = new Date();
    Object.prototype.toString.call(now); // "[object Date]"

When you concatenate, if `valueOf` returns a primitive type, it doesn't need to call `toString` as it can simply concatenate those two values.

If it doesn't return a primitive type, it will call `toString` until it gets a string, and proceed to do a string concatenation with it.

#Explaining some idiocy

I've seen this image pop up before and it may seem like pure idiocy, but following the above information, it makes sense!

![potato][3]

When it calls `dis.call(5)` it actually returns a wrapper type variable, instead of a non-wrapper type variable, which makes it extensible, so properties (`wtf`) can be added to the variable.

So while the code can add and change properties, `valueOf` still returns the `[[PrimitiveValue]]` of `5`.

However, when the increment operator is called, the variable is turned into an normal value type with an updated value.

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible
[2]: http://es5.github.io/spec.html#x15-toc
[3]: http://i.stack.imgur.com/LRjHf.jpg
