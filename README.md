# YDKJS Summary

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


# Chapter 3: Natives