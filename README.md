# YDKJS Summary

# Chapter 1: Types

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

# Chapter 2: Values
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


# Chapter 3: Natives

Defining an array with undefined values instead of empty slots:
```javascript
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

- JavaScript provides object wrappers around primitive values, known as natives (String, Number, Boolean etc.)
- Object wrappers give the values access to behaviors appropriate for each subtype (i.e. `String#trim()` and `Array#concat(..)`)

# Chapter 4: Coercion

## Explicit vs Implicit

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

Comparing `null` to `undefined`:
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

Comparing objects to non-objects:
The ES5 spec says in clauses 11.9.3.8-9:
 1  If Type(x) is either String or Number and Type(y) is Object, return the result of the comparison x == ToPrimitive(y).
 2  If Type(x) is Object and Type(y) is either String or Number, return the result of the comparison ToPrimitive(x) == y.


 ```javascript
var a = 42;
var b = [ 42 ];

a == b; // true
 ```

Comparing Booleans:
1. If Type(x) is Boolean, return the result of the comparison ToNumber(x) == y.
2.  If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).


```javascript
var x = true;
var y = "42";

x == y; // false, since ToNumber(true) = 1
```

Falsy comparisons:
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

**To effectively avoid issues with such comparisons, here’s some heuristic rules to follow:**
 •  If either side of the comparison can have true or false values, don’t ever, EVER use == .
 •  If either side of the comparison can have [] , "" , or 0 values, seriously consider not using == .

The question of == versus === is really appropriately framed as: should you allow coercion for a comparison or not?


# Chapter 5: Grammar

Expression Side Effects:
```javascript
var a = 42;

a++;    // 42
a;      // 43

++a;    // 44
a;      // 44
```

## Operator Precedence:
- `&&` has a higher precedence than `==`
- `&&` is more precedent than `||`

### Short Circuited:
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
