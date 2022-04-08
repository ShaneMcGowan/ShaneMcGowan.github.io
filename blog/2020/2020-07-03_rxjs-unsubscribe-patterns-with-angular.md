---
title: RxJS Unsubscribe patterns with Angular
published: true
description: A nice pattern for handling unsubscribing from observables in Angular when using RxJS and some not so nice patterns also.
tags: 
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/pp66u74mp2d97b452r99.png
---

RxJS Observables are classsss (if you know how to use them) but there are quite a few common issues people encounter. If you don't use them correctly you can end up with memory leaks. If you don't use them efficiently you can end up with a lot of unnecessarily verbose code.

# How to get memory leaks
In RxJS, each time you `subscribe` to an `Observable` or `Subject`, you then need to `unsubscribe`. If you don't `unsubscribe`, the `subscription` will exist in memory and any time the subject emits a value, the logic in the subscription will run.

```javascript
export class MemoryLeakComponent {

  // Our subject that we will subscribe to
  subjectA$: Subject<number> = new Subject<number>();

  constructor(){
    // Create your memory leaks
    this.subjectA$.subscribe(value => { 
      // logic goes here ...
    });
    this.subjectA$.subscribe(value => {
    // logic goes here ...
    });
    this.subjectA$.subscribe(value => {
    // logic goes here ...
    });
    this.subjectA$.subscribe(value => {
    // logic goes here ...
    });
  }

}

```
When you navigate away from this component and then navigate back, the subscriptions still exist and 4 more subscriptions will be added upon each return to this component. If you had, for example, subscribed to a `Subject` from an injected service, the logic in your subscription would run in the background any time a value was emitted from the `Subject`, even though the instance of the component it was relating to no longer exists. This can lead to memory leaks and performance issues.

To avoid this, you need to `unsubscribe` which comes with it's own caveats.

# How to end up with unnecessarily verbose code

```javascript
export class VerboseSubscriptionComponent implements OnDestroy {

  // Our subject that we will subscribe to
  subjectA$: Subject<number> = new Subject<number>();

  // The references we will use for our subscriptions;
  subscriptionA: Subscription;
  subscriptionB: Subscription;
  subscriptionC: Subscription;
  subscriptionD: Subscription;

  constructor(){
    // Create our verbose subscriptions
    this.subscriptionA = this.subjectA$.subscribe(value => { 
      // logic goes here ...
    });
    this.subscriptionB = this.subjectA$.subscribe(value => { 
      // logic goes here ...
    });
    this.subscriptionC = this.subjectA$.subscribe(value => { 
      // logic goes here ...
    });
    this.subscriptionD = this.subjectA$.subscribe(value => { 
      // logic goes here ...
    });
  }

  ngOnDestroy(){
    // Manually unsubscribe from each subscription
    this.subscriptionA.unsubscribe();
    this.subscriptionB.unsubscribe();
    this.subscriptionC.unsubscribe();
    this.subscriptionD.unsubscribe();
  }
}
```
This solves the memory leak issue but leaves room for error and is very verbose if we have a large amount of subscriptions. If at any point one of our `subscripton` variables is unsubscribed from or if it is never actually assigned to, when `unsubscribe()` is called in `ngOnDestroy()` it will error and break our app.

Below are some solutions that completely avoid this issue.

# A solution involving (wo)man's best friend; LOOPS!
We can create an `Array<Subscription>` to store all our subscriptions. Then we itterate over the array in the `ngOnDestroy()` method. With this solution, our code is much less verbose, we don't need to worry about keeping track of individual subscriptions and which one has or hasn't been unsubscribed from. 

```javascript
export class LoopSubscriptionComponent implements OnDestroy {
  // Our subject that we will subscribe to
  subjectA$: Subject<number> = new Subject<number>();

  // An array to hold our subscriptions
  subscriptions: Array<Subscription> = new Array<Subscription>()

  constructor(){
    // Add our subscription to
    this.subscriptions.push(
      this.subjectA$.subscribe(value => { 
        // logic goes here ...
      })
    );
  }

  ngOnDestroy(){
    // We loop our subscriptions array and unsubscribe
    this.subscriptions.forEach(subscription => subscription.unsubcribe());
  }

}
```

This is the solution I used to use when I thought I was really smart and cool. I have since been humbled and use the below, superior, solution.

# A better solution involving the `takeUntil()` pipeable operator 
Using the `takeUntil()` pipeable operator, we no longer need to call `unsubscribe()` on each individual subscription. This operator takes an observable as a paramater, and when a value is emitted from that observable, it will gracefully close the subscription is is applied to.
```javascript
export class TakeUntilComponent implements OnDestroy {
  // Our magical observable that will be passed to takeUntil()
  private readonly ngUnsubscribe$: Subject<void> = new Subject<void>();

  // Our subject that we will subscribe to
  subjectA$: Subject<number> = new Subject<number>();

  constructor() {
    this.subjectA$
      .pipe(takeUntil(this.ngUnsubscribe$))
      .subscribe(value => {
      // logic goes here ...
    });
  }

  ngOnDestroy(){
    // Emit a value so that takeUntil will handle the closing of our subscriptions;
    this.ngUnsubscribe$.next();
    // Unsubscribe from our unsubscriber to avoid creating a memory leak
    this.ngUnsubscribe$.unsubscribe();
  }

}
```

# Outro
Hopefully you learned something new here! I am by no means an expert but I write this hoping you can skip the first 3 itterations of learning and jump straight into the best pattern to work with (best being my opinion ofcourse).

If you know of a better pattern or solution, or if there are any issues with what I have written above, please let me know!

And most importantly, have a nice day ♥