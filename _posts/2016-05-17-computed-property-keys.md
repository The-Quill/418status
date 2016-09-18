---
layout: post
title:  "Computed property keys and overloading ToPrimitive"
date:   2016-06-15 13:46:00 +1000
comments: true
categories: code javascript down-the-rabbit-hole
---

In a [previous blog post][javascripts type system], I described how [`Symbol.toPrimitive`][toPrimitive] controls the flow of operation for certain operations on non-primitive types.

To follow up in this blog post, I'll describe how `toPrimitive` can be overloaded using computed property keys, what computed property keys are, and a little bit on what [`Symbol`s][Symbol] are.

Previously in JavaScript, you couldn't programmatically assign the values of your object keys in the object initializer syntax, but in ECMAScript 2015, computed property keys were added, allowing you to programmatically assign property keys with the `[expression]` syntax.

A good example of this can be found from [the docs][da docs]:

    var i = 0;
    var a = {
        ["foo" + ++i]: i,
        ["foo" + ++i]: i,
        ["foo" + ++i]: i
    };
    console.log(a.foo1); // 1
    console.log(a.foo2); // 2
    console.log(a.foo3); // 3

Using this new syntax we can do this:

    var someFunc = function(){
        console.log("cool!");
    };

    var someObject = {
        [Symbol.toPrimitive]: someFunc
    }

or using another "shorthand" syntax addition in ECMAScript 2015:

    var someObject = {
        [Symbol.toPrimitive](){
            console.log("cool!");
        }
    }


By default, the `Symbol.toPrimitive` function (property) would look something like this:

    function toPrimitive(hint){
        if (hint === 'string'){
            return this.toString();
        } else if (hint === 'number') {
            return Number(this);
        } else {
            return this.toString();
        }
    }

However, let's say we wanted a special type that returned a JSON equivalent for string conversions, and a length count for numbers.

We could do that like this:

    var stubObject = {
        [Symbol.toPrimitive](hint){
            if (hint === 'number'){
                return Object.keys(this).length;
            } else {
                return JSON.stringify(this);
            }
        }
    }

and then the way to "_impose_" that onto an existing object would be like this:

    function customObject(source){
        var stubObject = {
            [Symbol.toPrimitive](hint){
                if (hint === 'number'){
                    return Object.keys(this).length;
                } else {
                    return JSON.stringify(this);
                }
            }
        }
        return Object.assign(stubObject, source);
    }

Which would make results like this:

    var testObj = customObject({
        cat: 'dog',
        dog: 'cat'
    });
    console.log(+testObj); // hint is number, 2
    console.log(String(testObj)); //hint is string, "{"cat": "dog", "dog": "cat"}"
    console.log(testObj + ""); //hint is default, "{"cat": "dog", "dog": "cat"}"

The use case on a feature like this is pretty slim, outside of something using JavaScript's type system to make a custom type or similar.

[javascripts type system]: http://418stat.us/2016/04/javascripts-type-system/
[Symbol]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol
[toPrimitive]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive
[da docs]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer
