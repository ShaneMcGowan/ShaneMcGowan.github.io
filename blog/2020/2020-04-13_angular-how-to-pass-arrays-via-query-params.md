---
title: Angular - How to pass arrays via Query Parameters
published: true
description: How to pass an array of values via query parameter in Angular.
tags: Angular, Angular11, QueryParam, QueryParameters
---

This is a quick guide on how to pass an array of values via query string in Angular. This is working in Angular 9+ as of 13/04/2020 but will most likely work just fine for any version from Angular 2+.

In the example below we will create an array of values in `ComponentA` and pass them to `ComponentB` via navigation. This method will work for a direct navigation to `ComponentB` via a link and also via programmatic navigation with the Angular router.

*The short version is: `JSON.stringify` your array and pass it as query param, thend `JSON.parse` it back into an array after navigation*

#### Component A - Passing the array
```javascript
export class ComponentA {
  
  // We need access to the Angular router object to navigate programatically
  constructor(private router: Router){}

  navigateWithArray(): void {
    // Create our query parameters object
    const queryParams: any = {};
    // Create our array of values we want to pass as a query parameter
    const arrayOfValues = ['a','b','c','d'];

    // Add the array of values to the query parameter as a JSON string
    queryParams.myArray = JSON.stringify(arrayOfVAlues);

    // Create our 'NaviationExtras' object which is expected by the Angular Router
    const navigationExtras: NavigationExtras = {
      queryParams
    };

    // Navigate to component B
    this.router.navigate(['/componentB'], navigationExtras);
  }

}
```

#### Component B - parsing the array
```javascript
export class ComponentB {
  // Where we will be storing the values we get from the query string
  arrayOfValues: Array<string>;

  // We need use the ActivatedRoute object to get access 
  // to information about the current route
  constructor(private activatedRoute: ActivatedRoute){
    
    // Get the query string value from our route
    const myArray = this.activatedRoute.snapshot.queryParamMap.get('myArray');

    // If the value is null, create a new array and store it
    // Else parse the JSON string we sent into an array
    if (myArray === null) {
      this.arrayOfValues = new Array<string>();
    } else {
      this.arrayOfValues = JSON.parse(myArray));
    }
  }

}
```

#### Enjoy

Psst... Follow me on twitter https://twitter.com/TheShaneMcGowan