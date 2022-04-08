---
title: Working with Enums in Angular components
published: true
description: How I use Enums in Angular 
tags: angular, enums, typescript, angular9
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/gck88a5vqpn6cqz5uqnc.jpg
---

This is a short guide on using Enums in an Angular application.
Below you will see how to:
 - Reference an Enum in your `component.html` templates 
 - Itterate over an enum using `*ngFor` and the `keyvalue` pipe
 - Conditionally show content using an Enum and `*ngIf`
 - Get all the values of an enum as an `Array<sting>`

# Intro 
*You can skip the next block of text if you just want to get the actual details (I respect that)*

I love using Enums in TypeScript as I am a terrible developer with a terrible memory and let my editor's auto complete do 80% of my job. Enums let you avoid using strings and having to remember what string values you used. They also allow you to lock down the values a property can be, so that a user can't just submit any value they want. So if you aren't using them, start using them, and if you are already using them, use them more, use them to a fault. Be the Enums you want to see in the world.

# Example use of an enum for those unfamiliar
```javascript
// This is our PropertyType enum
// We give each value a string value otherwise PropertyType.House would return 0, Property.Apartment would return 1, and so on
enum PropertyType {
  House = 'House',
  Apartment = 'Apartment',
  Flat = 'Flat',
  Studio = 'Studio'
}

// Our class which has a 'type' property.
// We want to lock type down to a set of predefined values
class Property {
  constructor(
    public monthlyPrice: number,
    public type: PropertyType
  ) { }
}

// Creates a property successfully
const property = new Property(1250, PropertyType.Apartment);

// Will display an error from our linter and in the Angular CLI
const property = new Property(1250, PropertyType.RANDOM_VALUE);

```

Enums can be used in your Angular templates. This is handy for situations like avoiding a hard coded set of values for a dropdown menu or displaying different content based on your component state. In my opinion, this pairs very nicely with reactive forms.
To make an Enum accessible in your Angular template you need to first store it in a variable in our component.ts file. For this example, we will create a simple reactive form to create a `Property`

```javascript
// Declaring our Enum. The Enum could also be imported from another file if needed 
enum PropertyType {
  House = 'House',
  Apartment = 'Apartment',
  Flat = 'Flat',
  Studio = 'Studio'
}

@Component({
  selector: 'app-property-component',
  templateUrl: './property.component.html'
})
export class PropertyComponent{

  // Make a variable reference to our Enum
  ePropertyType = PropertyType;

  // Build our form
  form: FormGroup;
  type: FormControl;
  price: FormControl; // This is only here to make the form look more l3g1t

  constructor(private formBuilder: FormBuilder){
    this.form = this.formBuilder.group({
      type: [null, [Validators.required]],
      price: [0, [Validators.required]]
    });

    this.type = this.form.controls.type as FormControl;
    this.price = this.form.controls.price as FormControl;
  }
}
```
Then in our `property.html` template, we can utilize our Enum as follows. We will use the `keyvalue` pipe. This takes our `PropertyType` enum (referenced as `ePropertyType`) and turns it into a `KeyValue` array, which we can iterate over with `*ngFor`. As we declared our `PropertyType` with a string value (ie. `Apartment = 'Apartment'`) the `KeyValue.key` and `KeyValue.value` will both contain the string value of the enum so we can use either.

We then can use our Enum as part of an `*ngIf` to conditionally display a message.
```html
<form [formGroup]="form">
  <!-- Property type form control -->
  <label for="propertyType">Property Type</label>
  <select id="propertyType" [formControl]="type">
    <option [ngValue]="null">Select one...</option>
    <option *ngFor="let type of ePropertyType | keyvalue" [ngValue]="type.value">{{type.value}}</option>
  </select>

  <!-- Property price form control -->
  <label for="propertyPrice">Property Price</label>
  <input id="propertyPrice" class="form-control" type="number" [formControl]="price" placeholder="Street...">

</form>

<!-- Conditionally display a message if a certain PropertyType is selected (or not selected) -->
<ng-container *ngIf="type.value !== ePropertyType.House">
Wouldn't you prefer a nice garden?
</ng-container>
```


# How to get an array of strings from an Enum
A nice little trick to get an array of strings _(`string[]`, or `Array<string>` if you're a cool boi like myself)_ from an Enum is to use use `Object.keys` and `Array.filter`

```javascript
// Declare your enum
enum PropertyType {
  House = 'House',
  Apartment = 'Apartment',
  Flat = 'Flat',
  Studio = 'Studio'
}

/* Get an array of keys, filtering out number values
*  as the enum object itself is as follows
*  { 0: 'House', 1: 'Apartment' ...}
*/
const propertyType: Array<string> = Object.keys(PropertyType).filter(key => isNaN(+key));
```

# Conclusion
Enums are lit, use them more often. 

Psst... follow me on twitter https://twitter.com/TheShaneMcGowan