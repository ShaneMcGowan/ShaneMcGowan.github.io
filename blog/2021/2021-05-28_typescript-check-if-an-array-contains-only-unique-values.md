---
title: TypeScript - Check if an array contains only unique values
published: true
description: Quick guide on how to check if an array in TypeScript contains only a unique value
tags: typescript, javascript, array, set
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s8sk83b0xrmaqtlsmjo0.png
---

## Intro
A cool trick in TypeScript is that you can use a `Set` to remove nonunique values from an array. Combine this with `Array.from` and you get a quick one liner to remove none unique values

```typescript
const unqiueValues = Array.from(new Set(YOUR_ARRAY_HERE))
```

## Step by step
```typescript
// Our array of non unique values
const values = [0,0,1,'a','a','b','c'];

// Create a `Set` from our array
// Sets can only contain unique values so duplicate will be removed for us
const uniqueSet = new Set(values);

// Create a new array from our set so we can use the Array.length property
const unqiueValues = Array.from(uniqueSet);

// Compare the length of the original `values` array and the new `uniqueValues` array
// If the lengths differ then a non unique value was removed by the set
// This means that the original array contained non-unique values
const isUnique = (values.length === unqiueValues.length)
```

## As a one liner
```typescript
const values = [0,0,1,'a','a','b','c'];
const unique = (Array.from(new Set(values)).length === values.length);
```

## As a function
```typescript
onlyUnique(values: Array<any>): boolean {
  return (Array.from(new Set(values)).length === values.length);
}
```
Now follow me on twitter plz https://twitter.com/theshanemcgowan
