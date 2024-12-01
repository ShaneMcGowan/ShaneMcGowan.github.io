---
__BLOG_TITLE__: Creating a Game Update Loop with requestAnimationFrame 
__BLOG_SUBTITLE__: Creating a Game Engine in TypeScript - Part 2
__BLOG_DESCRIPTION__: In this blogpost, I discuss how to create a game update loop using requestAnimationFrame to create a game experience that is consistent across both low and high power devices. This is the second in a series of posts where I discuss creating a Game Engine from scratch with TypeScript and HTML5 Canvas. 
__BLOG_URL__:
__BLOG_IMAGE_URL__:
---

# Create a Game Update Loop with requestAnimationFrame

## What is a Game Update Loop
Our game update loop is what happens and in what order per frame of our game.
A game update loop should behave in a consistent and predictable way.

Our game update loop currently looks like the follow (this may, and probably will change, in future but it's a good example for now).

We have two `frame` functions. One on the `Client` which calculates our `delta` and performs more Client specific tasks and one on our `Scene` which is concerned purely with game logic. We will discuss the `Client` one later, but for now just know it passes a `delta` value.

```JavaScript
frame(delta: number): void {
  
  // Run the `onAwake` function for all SceneObjects that have `flags.awake` set to `false`.
  // This will only run once per SceneObject during it's existence 
  this.awake();

  // Render the background to an in memory canvas
  this.background(delta);

  // Run the `onUpdate` function for all SceneObjects that have `flags.update` set to `true`.
  // This will run every single frame
  this.update(delta);

  // Run the `onRender` function for all SceneObjects that have `flags.render` set to `true`.
  // All renders will be done to an in memory canvas
  // This will run every single frame
  this.render(delta);

  // using either a custom user defined renderer or the default renderer, the in memory canvas will be rendered to screen.
  if (this.customRenderer) {
    this.customRenderer(this.renderingContext);
  } else {
    defaultRenderer(this);
  }
  
  // all objects that were marked to be destroyed throughout the scene will now be cleaned up
  this.destroy(delta);
}
```

## What is a `delta` value?

As you would have noticed, we pass the `delta` value into most stages of our game loop. `delta` is the amount of time that has passed since our previous frame ran. This `delta` value will be used heavily throughout are game logic to ensure we have a consistent game experience across multiple devices.

## What do we need a `delta` value?

We could create our game without a delta value, and our game would run fine, but our physics and other calculations would be tied to the FPS of the machine we are running on, meaning that running our game on different specs of devices would exhibit different results.

For example, for the following devices running at the following FPS:
```
Device 1: 30 FPS
Device 2: 60 FPS
Device 3: 120 FPS
```

If our logic says to move by 1 tile per frame, after 1 second our 3 devices would be in completely different states.
```
Device 1 (30 FPS): Moved by 30 
Device 2 (60 FPS): Moved by 60 
Device 3 (120 FPS): Moved by 120 
```

Then after another 1 second
```
Device 1 (30 FPS): Moved by 60 
Device 2 (60 FPS): Moved by 120 
Device 3 (120 FPS): Moved by 240 
```

This isn't ideal as we want everyone to have an equivalent experience. This issue has been seen even in high profile AAA games such as Fallout 4 by Bethesda (https://www.youtube.com/watch?v=r4EHjFkVw-s&ab_channel=SolarBear)

Using a `delta` avoids this issue providing a (mostly) consistent experience across different device types. With a delta value, you would decide how many tiles you want to move per second, e.g. 60 and then multiply that by value by our delta to determine what fraction of our desired speed to move by this frame. After 1 second we will have moved by 60 tiles no matter if we were running at 10fps or 10000fps.

## `requestAnimationFrame` and getting our `delta` value

Our game loop is based around using `requestAnimationFrame`. `requestAnimationFrame` is provided by our browser and will run a passed callback function (our frame) before the next time the browser repaints.
`requestAnimationFrame` passes a single argument to our callback which is the timestamp of when the last frames rendering completed. We can then compare this timestamp to the previous frame’s timestamp to see how long has passed since the last frame. This is called our `delta`. This `delta` will be used throughout our `onUpdate` functions and is crucial for performing things such as physics calculations correctly.

Calls to `requestAnimationFrame` are none blocking which allows us to still respond to browser events and user Inputs. If we were to create a gameloop with something like `while(true) { }` we would block up the main thread and lose access to these things, and as we all _should_ know, you __never__ block the main thread.

## Calculating our `delta` value
We calculate our `delta` value as follows where the passed value `timestamp` is the timestamp of when the last frames rendering completed. This value comes from `requestAnimationFrame`.

This is done in our `Client` `frame` function we mentioned earlier.
```JavaScript
private frame(timestamp: number): void {
  ...

  this.setDelta(timestamp);

  ...

  // Queue up next frame
  window.requestAnimationFrame(this.frame.bind(this));
}
```

Where `setDelta` is calculated as below.

```JavaScript
private setDelta(timestamp: number): void {
  
  // find the difference in milliseconds between our two timestamps
  // we then divide by 1000 so that we can do our game logic based on seconds 
  this.delta = (timestamp - this.lastRenderTimestamp) / 1000;
  
  // store for next frame's calculation
  this.lastRenderTimestamp = timestamp;
}
```

We divide our delta value by `1000` so that we can do our game logic based on `seconds` rather than `milliseconds`.

For example: If we want to move by `60` tiles per second, we can set our movement speed to `60` and multiply by the `delta`. If we didn't divide by `1000` we would need to say how much we want to move per millisecond which would be `0.006` which is a bit more tedious to work with.

## requestAnimationFrame additional cases

### Providing a timestamp value for the very first frame
For the very first frame of our game, we need to perform an additional call to `requestAnimationFrame` to get a timestamp to use as our basis.

We can do this by creating a `firstFrame` function which will set a starting timestamp then start our game loop afterwards.
```JavaScript
private firstFrame(timestamp: number): void {
  this.setDelta(timestamp);

  window.requestAnimationFrame(this.frame.bind(this));
}
```

To start out gameloop in this fashion, we will call the following.
```JavaScript
window.requestAnimationFrame(this.firstFrame.bind(this));
```

This avoids our first frame having a huge delta compared to all of our other frames, which could potentially lead to unexpected behaviour.

### Tab becoming inactive / tabbing in and out
Modern browsers will pause execution of our game if it is in a background tab. This is not ideal if you want your game to keep running while tabbed out but browsers will start to throttle the game and potentially mess things up anyway.
While it would be ideal to keep our game running in the background, our engine is going to just accept this limitation for now. I may come back to this in the future but there are currently no plans.

Due to this behaviour, when our frame is interrupted by our tab becoming inactive (be it tabbing out or our device going to sleep), we need to react to the tab becoming active again and setting an appropriate delta, similar to how we do for our very first frame.

```JavaScript
// this is the event we want to listen to
document.addEventListener('visibilitychange', (event) => {
  if (document.visibilityState === 'visible') {
    // frame is active
    // we want to set a flag that our next frame should calculate a new timestamp for delta
    this.flags.frame.interrupted = false;
  } else {
    // frame is inactive
  }
});
```

We don't currently do anything when the frame becomes inactive but this could be a good point to save your game state.

Similar to our `firstFrame` function, we will use `requestAnimationFrame` get a new timestamp.
```JavaScript
private interruptedFrame(timestamp: number): void {
  // create a new updated timestamp to base delta off
  this.lastRenderTimestamp = timestamp;

  // remove flag for frame interrupted
  this.flags.frame.interrupted = false;

  // recall our frame but with our new timestamp
  window.requestAnimationFrame(this.frame.bind(this));
}
```

Then at the start of our `Client` `frame` function before any frame logic runs.

```JavaScript
private frame(timestamp: number): void {
  // flag has been set, we need to get a new timestamp
  if (this.flags.frame.interrupted) {
    
    // call our interruptedFrame function to set a new lastRenderTimestamp
    window.requestAnimationFrame(this.interruptedFrame.bind(this));
    
    // cancel this frame, interruptedFrame will kick off a new one
    return;
  }

  ...

}
```

Just to demonstrate why we require both the `firstFrame` and `interruptedFrame` edgecases, see the below case:

#### Without frame interrupted flag
```bash
[timestamp] 5817.2
[delta] 0.008099999999999455
[timestamp] 5825.7
[delta] 0.0085
tab is inactive
tab is active
[timestamp] 10600.4
[delta] 4.7747 // MASSIVE delta
[timestamp] 10609.1
[delta] 0.008700000000000728
```
The last frame before the tab went inactive had a timestamp of `5825.7` with a delta of `0.0085`.
The first frame after the tab became active has a timestamp of `10600.4` with a __MASSIVE__ delta of `4.7747`.
This is `561` times larger than our previous delta and has the potential to mess up our game's logic.

#### With frame interrupted flag
```bash
[timestamp] 9623.8
[delta] 0.008399999999999637
[timestamp] 9632.2
[delta] 0.008400000000001455
tab is inactive
tab is active
frameInterrupted
[timestamp] 13419.6
[delta] 0.008
[timestamp] 13428.2
[delta] 0.008600000000000364
```
The last frame before the tab went inactive had a timestamp of `9632.2` with a delta of `0.008400000000001455`
The first frame after the tabe became active has a timestamp of `13419.6` with a delta of `.008`.
This delta is roughly the length of our average delta, meaning our game logic is safe from unwanted side effects. As far as our game knows, we were here all along and never tabbed out.