# YDKJS Summary

## Types and Grammar
## Chapter 1: Types

- JavaScript has 7 built-in types:
1. null
2. undefined
3. boolean
4. number
5. string
6. object 
7. symbol

- Variables don't have types, but the values in them do.
- Undefined and undeclared are not the same thing
  - `undefined` is a value that a declared variable can hold
  - "undeclared" means a variable has never been declared
  - JS conflates these two terms in
    1. Error messages = `ReferenceError: a is not defined`
    2. Return values of `typeof` which is undefined for both 

## Chapter 2: Values
- Arrays are simply numerically indexed collections of any type
- Strings are similar but are immutable (easier to work with arrays)
- Numbers in JS include both integers and floating-point values
- Special values defined within primitive types
  - `null` and `undefined` have just one value = themselves
  - `undefined` is the default value in any variable or property if no other value is present
  - `void` lets you create `undefined` from any other value
  - Numbers include several special values like NaN (can be thought of as "invalid num,ber" i.e. +infinity, -infinity and -0)
  - Simple scalar primitives are assigned/passed by value-copy but compound values are assigned/passed by reference-copy.
  - References are not like references/pointers in other languages they are never pointed at other variables/references only at the underlying values


## Chapter 3: Natives

Defining an array with undefined values instead of empty slots:
```javascript
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

- JavaScript provides object wrappers around primitive values, known as natives (String, Number, Boolean etc.)
- Object wrappers give the values access to behaviors appropriate for each subtype (i.e. `String#trim()` and `Array#concat(..)`)

## Chapter 4: Coercion

### Explicit vs Implicit

```javascript
var a = 42;
var b = a + ""; // Implicit

car c = String(a) // Explicit
```

**Falsy values:**
- undefined
- null
- false
- +0, -0 and NaN
- ""

Explicit Coercion:
```javascript
var a = "42"
var b = "42px"
Number(a); // 42
parseInt(a); // 42
Number(b); // NaN
parseInt(b); // 42
```
- Never use `parseInt(..)` with a non-string value. (creates an implicit string conversion first)
- Prior to ES5 if you didn't pass a second argument to indicate which numeric base (aka radix) to use \
for interpreting the numeric string arguments `parseInt(..)` would make a guess based on first character

- Comparing implicit and explicit coercion:

```javascript
var a = {
  valueOf: function () { return 42; },
  toString: function () { return 4; },
}
a + ""; // "42"
String(a); // "4" since String(..) invokes toString directly
```

Operators `||` and `&&`:
```javascript
var a = 42;
var b = "abc";
a || b; // 42
a && b; // "abc"
c || b; // "abc"
c && b; // null

// Note: In languages like C and PHP, those expressions result in true or false,
// but in JS (and Python and Ruby, for that matter!), the result comes from the values themselves.

// Understanding ||:
// For the || operator, if the test is true , the || expression results in the value of the first operand ( a or c ).
// If the test is false , the || expression results in the value of the second operand ( b).
// e.g. a || b => a ? a: b

// Inverse case for &&
// e.g. a && b => a ? b : a
```

`ToNumber` coercion:
In the ES5 spec, clauses 11.9.3.4-5 say:
 1  If Type(x) is Number and Type(y) is String, return the result of the comparison x == ToNumber(y).
 2  If Type(x) is String and Type(y) is Number, return the result of the comparison ToNumber(x) == y.

```javascript
var a = 42;
var b = "42";
a === b; // false
a == b; // true, because 42 == Number(b)
```

Consider:

```javascript
var a = "42";
var b = true;
a == b; // false
// 1.  "42" == Number(b)
// 2. "42" == 1
// 3. Number(a) == 1
// 4. 42 == 1
// 5. false
```

### Comparing `null` to `undefined`:
Quoting the ES5 spec, clauses 11.9.3.2-3:
 1  If x is null and y is undefined, return true.
 2  If x is undefined and y is null, return true.

Instead of:

```javascript
var a = doSomething();

if (a === undefined || a === null) {
    // ..
}
```

We can use the more readable (implicit coercion):

```javascript
var a = doSomething();

if (a == null) {
    // ..
}
```

### Comparing objects to non-objects:
The ES5 spec says in clauses 11.9.3.8-9:
 1  If Type(x) is either String or Number and Type(y) is Object, return the result of the comparison x == ToPrimitive(y).
 2  If Type(x) is Object and Type(y) is either String or Number, return the result of the comparison ToPrimitive(x) == y.


 ```javascript
var a = 42;
var b = [ 42 ];

a == b; // true
 ```

### Comparing Booleans:
1. If Type(x) is Boolean, return the result of the comparison ToNumber(x) == y.
2.  If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).


```javascript
var x = true;
var y = "42";

x == y; // false, since ToNumber(true) = 1
```

### Falsy comparisons:
```javascript
"0" == null;            // false
"0" == undefined;       // false
"0" == false;           // true -- UH OH!
"0" == NaN;             // false
"0" == 0;               // true
"0" == "";              // false

false == null;          // false
false == undefined;     // false
false == NaN;           // false
false == 0;             // true -- UH OH!
false == "";            // true -- UH OH!
false == [];            // true -- UH OH!
false == {};            // false

"" == null;             // false
"" == undefined;        // false
"" == NaN;              // false
"" == 0;                // true -- UH OH!
"" == [];               // true -- UH OH!
"" == {};               // false

0 == null;              // false
0 == undefined;         // false
0 == NaN;               // false
0 == [];                // true -- UH OH!
0 == {};                // false
```

Crazy ones:
```javascript
[] == ![];      // true
// It explicitly coerces to a boolean using the ToBoolean rules (and it also flips the parity).
// parity). So before [] == ![] is even processed, it’s actually already translated to [] == false
```

```javascript
2 == [2];       // true
"" == [null];   // true

// [2] will become "2" which becomes 2
// [null] will become ""
```

**To effectively avoid issues with such comparisons, here’s some heuristic rules to follow:**\
 •  If either side of the comparison can have true or false values, don’t ever, EVER use == .
 •  If either side of the comparison can have [] , "" , or 0 values, seriously consider not using == .

The question of == versus === is really appropriately framed as: should you allow coercion for a comparison or not?


## Chapter 5: Grammar

Expression Side Effects:
```javascript
var a = 42;

a++;    // 42
a;      // 43

++a;    // 44
a;      // 44
```

### Operator Precedence:
- `&&` has a higher precedence than `==`
- `&&` is more precedent than `||`

#### Short Circuited:
For both `&&` and `||` operators, the right hand operand will not be evaluated if the left hand operand
is sufficient to determine the outcome of the operation.

Examples:
If `opts` is unset or not an object. `opts.cool` won't be evaluated. (acts as a guard).
```javascript
function doSomething(opts) {
    if (opts && opts.cool) {
        // ..
    }
}
```

Check for `opts.cache` first and if present we don't call `primeCache()`.
```javascript
function doSomething(opts) {
    if (opts.cache || primeCache()) {
        // ..
    }
}
```

### Tighter Binding:
Consider:
`a && b || c ? c || b ? a : c && b : a`

Which can be evaluated as:
`(a && b || c) ? (c || b) ? a : (c && b) : a`

Because:
`&&` is more precedent than `||`, and  `||` is more precedent than `? :`


### Associativity:
- `&&` and  `||` are left-associative. (refers to grouping not evaluation)

- `? :` is right-associative.
- Consider:
  - `a ? b : c ? d : e;`
  - Evaluates like:` a ? b : (c ? d : e)`
- Another example of right-associativity (grouping) is the `=` operator
  - E.g. 
  ```javascript
  var a, b, c;
  a = b = c = 42;
  // a = (b = (c = 42))
  ```

Last example:

```javascript
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d;      // 42
```

`((a && b) || c) ? ((c || b) ? a : (c && b)) : a`

Let’s solve it now:
1.  (a && b) is "foo" .
2.  "foo" || c is "foo" .
3.  For the first ? test, "foo" is truthy.
4. (c || b) is "foo" .
5.  For the second ? test, "foo" is truthy.
6.  a is 42 .

### Errors
Grammar run time errors
Example:

`var a = /+foo/; // Error - due to invalid regex!`

ES5's `strict` mode defines even more early errors:
1. Function parameter names cannot be duplicated:
```javascript
function foo(a,b,a) { }                 // just fine

function bar(a,b,a) { "use strict"; }   // Error!
```

2. Having more than one property of the same name
```javascript
(function(){
    "use strict";

    var a = {
        b: 42,
        b: 43
    };          // Error!
})();
```

### Temporal Dead Zone (TDZ):
- A variable reference cannot yet be made, because it hasn't reached its required initialization.
 ```javascript
{
    a = 2;      // ReferenceError!
    let a;
}
 ```

 Interestingly, while `typeof` has an exception to be safe for undeclared variables (see Chapter 1 ), no such safety exception is made for TDZ references:
```javascript
{
    typeof a;   // undefined
    typeof b;   // ReferenceError! (TDZ)
    let b;
}
```

### Function arguments:

Outer reference `b` will cause an error. However, `a` is fine since by that time it's past the TDZ for parameter `a`.
```javascript
var b = 3;

function foo( a = 42, b = a + b + 5 ) {
    // ..
}
```
When using ES6's default parameter values, the default value is applied if uou omit an argument, or you pass an `undefined` value in it's place.

```javascript
function foo( a = 42, b = a + 1 ) {
    console.log( a, b );
}

foo();                  // 42 43
foo( undefined );       // 42 43
foo( 5 );               // 5 6
foo( void 0, 7 );       // 42 7
foo( null );            // null 1 (since null is coerced to 0)
```

## try..finally
- `Finally` clause always runs (no matter what) and always runs right after the `try (and catch` if present)
```javascript
function foo() {
        try {
                return 42;
        }
        finally {
                console.log( "Hello" );
        }

        console.log( "never runs" );
}

console.log( foo() );
// Hello
// 42
```
- The return 42 runs right away, which sets up the completion value from the foo() call.
- This action completes the try clause and the finally clause immediately runs next.
- Only then is the foo() function complete, so that its completion value is returned back for the console.log(..) statement to use.


 The exact same behavior is true of a throw inside try :
 ```javascript
 function foo() {
        try {
                throw 42;
        }
        finally {
                console.log( "Hello" );
        }

        console.log( "never runs" );
}

console.log( foo() );
// Hello
// Uncaught Exception: 42
```

If an exception is thrown (accidental or intentionally) inside a `finally` clause, it will override the primary completion of the function.

```javascript
function foo() {
        try {
                return 42;
        }
        finally {
                throw "Oops!";
        }

        console.log( "never runs" );
}

console.log( foo() );
// Uncaught Exception: Oops!
```

A return inside a finally has the special ability to override a previous return from the try or catch clause, but only if return is explicitly called.

```javascript
function foo() {
        try {
                return 42;
        }
        finally {
                // no `return ..` here, so no override
        }
}

function bar() {
        try {
                return 42;
        }
        finally {
                // override previous `return 42`
                return;
        }
}

function baz() {
        try {
                return 42;
        }
        finally {
                // override previous `return 42`
                return "Hello";
        }
}

foo();   // 42
bar();   // undefined
baz();   // Hello
```

## `this` & Object Prototypes

### Chapter 1: `this` or That?
#### Reviewing what `this` isn't
`this` is neither a reference to the function itself, nor is it a reference to the function’s lexical scope.

`this` is actually a **binding** that is made when a **function is invoked**, and what it **references** is determined entirely by the **call-site** where the function is called.

### Chapter 2: `this` All Makes Sense Now!

#### Call-Site
The location in code where a function is called (not where it’s declared).

```javascript
function baz() {   
// call-stack is: `baz`    
// so, our call-site is in the global scope    
console.log( "baz" );    
bar(); // <-- call-site for `bar`
}
function bar() {   
// call-stack is: `baz` -> `bar`    
// so, our call-site is in baz    
console.log( "bar" );    
foo(); // <-- call-site for `foo`
}
function baz() {   
// call-stack is: `baz` -> `bar` -`foo`    
// so, our call-site is in bar    
console.log( "foo" );    
baz(); // <-- call-site for `baz`
}
```

#### Explicit Binding
We create a function bar() which, internally, manually calls foo.call(obj),
thereby forcibly invoking foo with obj binding for this . No matter how you later
invoke the function bar, it will always manually invoke foo with obj.
This binding is both explicit and strong, so we call it hard binding.

```javascript
function foo() {    
  console.log( this.a );
}
var obj = {a: 2};
var bar = function() { 
	foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2
// hard-bound `bar` can no longer have its `this` overridden
bar.call( window ); // 2
```

Most typical way to wrap a function with a hard binding:
```javascript
function foo(something) {    
  console.log( this.a, something );
}
var obj = {a: 2};
var bar = function() { 
	foo.apply( obj, arguments );
};
var b = bar(3); // 2 3
console.log( b); // 5
```
Hard binding is provided with a built in utility as of ES5. `Function.prototype.bind`:
```javascript
function foo(something) {    
  console.log( this.a, something );
}
var obj = {a: 2};
var bar = foo.bind(obj);
var b = bar(3); // 2 3
console.log( b); // 5
```
#### Determining this
1. Is the function called with new ( **new binding** )? If so, `this` is the newly constructed object.
`var bar = new foo()`
2. Is the function called with call or apply ( **explicit binding** ), even hidden inside a bind hard binding ? If so, `this` is the explicitly specified object.
`var bar = foo.call( obj2 )`
3. Is the function called with a context ( **implicit binding** ), otherwise known as an owning or containing object? If so, `this` is that context object.
var bar = obj1.foo()
4. Otherwise, default the `this` ( **default binding** ). If in strict mode, pick undefined , otherwise pick the global object.
`var bar = foo()`


#### Binding Exceptions
However, there’s a slight hidden “danger” in always using null when you don’t care about the this binding.
If you ever use that against a function call (for instance, a third-party library function that you don’t control),
and that function does make a this reference, the default binding rule means it might inadvertently reference (or worse, mutate!) the global object ( window in the browser).
Obviously, such a pitfall can lead to a variety of bugs that are very difficult to diagnose and track down.

#### Safer this
- User `Object.create(null)` similar to `{}` but without the delegation to Object.prototype

```javascript
function foo(a,b) { 
        console.log( "a:" + a + ", b:" + b );
}
// our DMZ empty object
var ø = Object.create( null );
// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3
// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

#### Lexical this
Arrow functions don't play by the same rules:
```javascript
function foo() {    
	// return an arrow function   
	return (a) => {        
		// `this` here is lexically inherited from `foo()`        
		console.log( this.a );   
		};
	}
	var obj1 = { a: 2};
	var obj2 = { a: 3};
	var bar = foo.call( obj1 );
	bar.call( obj2 ); // 2, not 3!
```
The arrow-function created in foo() lexically captures whatever foo() s this is at its call-time.
The lexical binding of an arrow-function cannot be overridden (even with new !).

### Chapter 3: Objects

#### Syntax
Literal object:

```javascript
var myObj = {
	key: value
	// ...
}
```

Constructed form:
```javascript
var myObj = new Object();
	myObj.key = value;
```

The only difference really is that you can add **one or more key/value pairs** to the **literal** declaration, whereas with ***constructed-form*** objects, you must add the properties **one by one**.

Extremely uncommon to use the "constructed form".

#### Types
There are 6 primitive types:\
string •  number •  boolean •  null •  undefined •  object

#### Built-in Objects
Object subtypes refereed to as built-in objects.
- String 
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

Note that, The primitive value "I am a string" is not an object, it’s a primitive literal and immutable value. To perform operations such as checking it's length etc. a String object is required.

Luckily, the language **automatically coerces** a string primitive to a **String object** when necessary, which means you almost never need to explicitly create the Object form.

Same sort of coercion happens for number literal and boolean counterparts.

`null` and `undefined` have no object wrapper form, only their primitive values. By contrast `Date` values can only be created with their constructed object form, as they have no literal form counterparts.

#### Contents
```javascript
var myObject = {
	a: 2;
};
myObject.a; //2
myObject["a"]; //2
```
The .a syntax is usually referred to as “property access,” whereas the ["a"] syntax is usually referred to as “key access.”

In objects, property names are always strings. If you use any other value besides a string (primitive) as the property, it will first be converted to a string. 

This even includes numbers, which are commonly used as array indexes, so be careful not to confuse the use of numbers between objects and arrays:

**Computed Property Names**
ES6 adds computed property names , where you can specify an expression, surrounded by a [ ] pair, in the key-name position of an object-literal declaration:

```javascript
var prefix = "foo";
var myObject = {
	[prefix + "bar"] : "hello",
	[prefix + "baz"] : "world",
};
myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

#### Arrays

Arrays are objects, so even though each index is a positive integer, you can also add properties onto the array.

```javascript
var myArray = ["foo", 42, "bar"];
myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```

Note that adding a new property did not change the length.

Be careful adding adding a new property where the property name looks like a number, it will end up instead as a numeric index.

```javascript
var myArray = ["foo", 42, "bar"];
myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

#### Duplicating Objects