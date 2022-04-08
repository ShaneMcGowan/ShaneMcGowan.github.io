---
title: TypeScript / JavaScript - Check if 2 arrays of primitives contain the same values
published: true
description: How to check if an array of primitives contains the exact same values (in any order) in JavaScript (and TypeScript) 
tags: javascript, typescript, array, primitive
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h9nbtgrlw4nf4m5yj1ye.jpg
---

More JavaScript / Typescript tricks

## Intro 

How to check if an array of primitives contains the exact same values (in any order) in JavaScript (and TypeScript). 

This works for primitives and object references but not the values of an object. (look up primitives in JavaScript)

This can be achieved by checking if all items of Array 1 are in Array 2 and if all items of Array 2 are in Array 1

## Walkthrough

```JavaScript
// array 2 contains every value of Array 1
const array2containsAll = array1.every(value => array2.includes(value));

// array 1 contains every value of Array 2
const array1containsAll = array2.every(value => array1.includes(value));

// if both are true then both arrays contain the same items
if(array1containsAll === true && array2containsAll === true){
  // arrays match
} else {
  // arrays don't match
}
```

## As a JavaScript function
```JavaScript
arraysMatch(array1, array2) {
  return (
    array1.every(value => array2.includes(value)) 
    && array2.every(value => array1.includes(value))
  );
}
```

## As a TypeScript function
(TypeScript is for cool kids btw)
TypeScript doesn't have a specific type for `primitives` so swap out `string` below for whatever type you require, or `any` if you don't care too much about type safety.
```TypeScript
arraysMatch(array1: Array<string>, array2: Array<string>): boolean {
  return (
    array1.every(value => array2.includes(value)) 
    && array2.every(value => array1.includes(value))
  )
}
```

Now follow me on twitter plz https://twitter.com/theshanemcgowan

