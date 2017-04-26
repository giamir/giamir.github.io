---
layout: post
permalink: /what-is-react-fiber
title: "What is React Fiber ?"
path: _posts/2017-04-23-what-is-react-fiber.md
time: 10
---

During the last months the interest of the community to the upcoming version of React has grown rapidly, especially after the awesome job [Lin Clark](https://twitter.com/linclark) did to explain how `React Fiber` works with [cartoons](https://www.youtube.com/watch?list=PLb0IAmt7-GS3fZ46IGFirdqKTIxlws7e0&v=ZCuYPiUIONs) at React Conf 2017.

So, what is `React Fiber` in a nutshell?

> **React Fiber** is an ongoing reimplementation of the React reconciler.

It's a ground-up rewrite of the current reconciliation algorithm which now has been retroactively called `React Stack` because it is based on recursion.

<small>DISCLAIMER: The following are personal notes and deductions after exploring the new `React Fiber` reconciler codebase. Some of the content can be wrong or not updated, please open a [pull request](https://github.com/giamir/giamir.github.io/edit/master/_posts/2017-04-23-what-is-react-fiber.md) if you notice any mistake. 🙏</small>

## What is the reconciler ?

When React first came out the most revolutionary feature was the `Virtual DOM` because it makes writing applications a lot easier. Instead of telling the browser exactly what it needs to do in order to update your UI, you would tell React how the next state of your application should look like and it would take care of everything in between.

> The **reconciler** is the part of React which contains **the algorithm used to diff one tree with another** to determine which parts need to be changed.

In the last years React has been refactored out so that *reconciliation* and *rendering* are separate phases. The `reconciler` does the work of computing which parts of a tree have changed; the `renderer` then uses that information to actually update the rendered app. `react-dom` and `react-native` are only two of the [many renderers](https://github.com/chentsulin/awesome-react-renderer) pluggable into React.

<small>You can find out more about the reconciliation process in the [Official React Documentation](https://facebook.github.io/react/docs/reconciliation.html).</small>

## Why rewrite the reconciler ?

Probably one of the main reasons is this historical problem we have in browsers:

> **The main thread is the same as the UI thread**.

Rendering the page, responding to user actions, computing JS, managing network activities, and manipulating the DOM is all handled by the browsers main thread. Although today some of these things can be moved to another thread safely and with relative ease using Workers, **only the main thread can change the DOM**.

In React applications when the state changes or the props update the `render()` function create a new tree of React Elements and then React runs the reconciliation algorithm to figure out how to efficiently update the UI to match the new tree.

The `React Stack` reconciler always processes the component tree **synchronously** in a single pass. This **prevents the main thread to do potential urgent work** until the recursive process has finished. If this computation runs while the user happen to type on a text input, your app can become **unresponsive**, resulting in **choppy frame rates** and **laggy input**.

The new `Fiber` reconciler main goal is to **split interruptible work in chunks** with the ability to **assign priority to different types of updates** so that the main thread could decide to pause the diff algorithm, potentially do some more urgent work, and continue from where it left at a later time.

As mentioned in the scheduling section of the [React Design Principles Documentation](https://facebook.github.io/react/contributing/design-principles.html#scheduling) `Fiber` will enable the following functionalities:

> If something is offscreen, we can delay any logic related to it. If data is arriving faster than the frame rate, we can coalesce and batch updates. We can prioritize work coming from user interactions (such as an animation caused by a button click) over less important background work (such as rendering new content just loaded from the network) to avoid dropping frames.

## The triangles demo
To better understand the benefits `React Fiber` can provide to our React applications Sebastian Markbåge who is considered the "tech lead" of the React Core Team built a useful fractals example. Check out the following video:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The Fiber Triangle demo now lets you toggle time-slicing on and off. Makes it much easier to see the effect. Thanks <a href="https://twitter.com/giamir">@giamir</a> for the PR! 🎉 <a href="https://t.co/qhsWUIyXPf">pic.twitter.com/qhsWUIyXPf</a></p>&mdash; Andrew Clark (@acdlite) <a href="https://twitter.com/acdlite/status/846456239693344769">March 27, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<small>Source code of the Triangles Demo can be found in the [Official Github React Repository](https://github.com/facebook/react/blob/master/fixtures/fiber-triangle/index.html).</small>

As you can see from the video, there are 2 different kind of updates going on in this demo.

One that make the triangles narrow and wide which has to happen every 16ms (60 FPS) in order for the animation to look smooth and one that updates the numbers contained inside every single dots which happen approximately every 1000ms.

This example is perfect for analysing a situation where we want to be able to assign different priorities to different types of updates. The *animation update* which make the triangles wide and narrow is more important than the *numbers* one. If the numbers are updated with some delay the user will probably not even notice it. In the other hand if the animation starts to dropping frame we would end up staring at a janky motion.

Let's use this example to check out how the `Fiber` reconciler works and how we can split work in chunks.

## A taste of React Fiber architecture

Andrew Clark's [React Fiber Architecture document](https://github.com/acdlite/react-fiber-architecture) is so good on explaining the idea behind `Fiber` implementation that I'll just quote it here:

> Fiber is a **reimplementation of the stack**, specialized for React components. You can think of a single fiber as a **virtual stack frame**.<br />
The advantage of reimplementing the stack is that you can **keep stack frames in memory** and execute them however (and whenever) you want. This is crucial for accomplishing the goals we have for **scheduling**.

And more in detail:

> A fiber is a JavaScript object that contains information about a component, its input, and its output. At any time, a component instance has at most two fibers that correspond to it: the **current**, flushed fiber, and the **work-in-progress** fiber.

And here it's how a very simplified version of a fiber (virtual stack frame) looks like:

```
{
  stateNode,
  child,
  siblings,
  return,
  alternate,
  type,
  key
}
```
<small>Digging through the properties of a fiber goes beyond the purpose of this post. <br />
If you are interested on understanding them I strongly recommend you to read the [document](https://github.com/acdlite/react-fiber-architecture) written by [Andrew Clark](https://twitter.com/acdlite). You can also play around with the [Fiber Debugger](https://github.com/facebook/react/tree/master/fixtures/fiber-debugger) developed by [Dan Abramov](https://twitter.com/dan_abramov) to visualise how `Fiber` works internally.</small>

![Animation Update Timeline](https://s3.eu-west-2.amazonaws.com/websitegiamir/triangles-demo-animation-update.png)

<small>Animation update timeline from the Triangles demo.</small>

`React Fiber` introduces phases in the reconciliation algorithm.

There is a first **render / reconciliation phase** where `Fiber` will build up a *working-in-progress* tree and figure out a list of the changes it needs to make in order to update the UI. It will **not** make any actual changes though at this stage.

This first phase is **interruptible** and we will see later in the example how this peculiarity becomes key on allowing high priority updates to jump ahead of low priority updates.

Then there is a **commit phase** where `Fiber` will actually make all the changes figured out in the previous phase to the DOM.

This second phase is **uninterruptible** because we do not want to end up with an inconsistent UI. Partial updates can be annoying to the user and `Fiber` avoids them.

`Fiber` will also call the lifecycle hooks and handle error boundaries as part of the commit phase.

### The triangles demo without time slicing (janky animation)

Let's try to figure out from the following timeline what causes the janky animation.

![Triangles Demo without Time Slicing](https://s3.eu-west-2.amazonaws.com/websitegiamir/triangles-demo-without-time-slicing.png)

<small>Animation and numbers updates timeline from the Triangles demo without time slicing.</small>

As we can see from the picture above when a *numbers update* is triggered `Fiber` will iterate through the whole tree looking for changes while building up the `work-in-progress tree`. This keeps the main thread busy for around `350ms`.

Prevent the main thread to manage the *animation updates* causes a really long frame and consequentially the effects the user will see is a janky motion.

This is happening because we are using `React Fiber` on compatibility mode with `React Stack`. All the updates are not prioritised and they are treated **synchronously**.

### The triangles demo with time slicing (smooth animation)

Now let's analyse how Fiber can defer updates to avoid dropping frames. This functionality of Fiber is also known as Time slicing or async setState.

![Triangles Demo with Time Slicing](https://s3.eu-west-2.amazonaws.com/websitegiamir/triangles-demo-with-time-slicing.png)

<small>Animation and numbers updates timeline from the Triangles demo with time slicing.</small>

This time we are using the new `unstable_deferredUpdates` API to update numbers:

```
ReactDOM.unstable_deferredUpdates(() => {
  this.setState(state => ({ seconds: (state.seconds % 10) + 1 }));
});
```

<small>A thing to keep in mind is that we are passing to `setState` an updater function rather than a plain object.
This is necessary in order to defers updates. More about functional setState can be found in [this article](https://medium.freecodecamp.com/functional-setstate-is-the-future-of-react-374f30401b6b).</small>

When a *numbers update* is triggered, as you can see from the picture, `React` just calls the `requestIdleCallBack` function and it does not even start to look for changes on the new tree.

**Numbers update is now considered low priority and it is therefore deferred**.

I guess you are wondering what is this [requestIdleCallback function](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback).

```
requestIdleCallback(callback);
```

> **requestIdleCallback** is a window method that provides a way to cooperate with the browser’s overall work schedule.

The callback argument we pass to `requestIdleCallback` is evaluated by the main thread as soon as it identifies an idle period. An idle period may be a few milliseconds between painting individual frames.

So `requestIdleCallback` gives to `Fiber` an efficient way to understand when the main thread has some "spare time" to do some work.

Focusing again on the triangle demo we can see on the picture a *chunk of the numbers update reconciliation phase*. The chunk of work starts when the idle callback event is fired: the browser is telling `Fiber` that it got some "spare time" to do work.

The browser also gives to `Fiber` a *time remaining value* which provides an estimate of the number of milliseconds remaining in the current idle period. When this time is over `Fiber` knows have to stop doing work and relinquish control back to the main thread which has something more important to do; in our case do an *animation update*.

Before passing control to the main thread `Fiber` makes sure to call `requestIdleCallback` again because it just paused the reconciliation phase and it still has work to do. It needs the main thread to come back to it as soon as it identifies a new idle period.

![Numbers Update Chunk](https://s3.eu-west-2.amazonaws.com/websitegiamir/triangles-demo-numbers-update.png)

<small>Numbers update chunk timeline from the Triangles demo with time slicing.</small>

The image above shows a chunk of the *numbers update reconciliation phase*. `Fiber` managed to update 18 dots out of 729 in this particular chunk of work. This suggests the work has been split in around 40 chunks of work allowing the main thread to handle the *animation update* and repainting 40 times.

This way of "collaborate" with the main thread in order to do work is known under the name of [Cooperative Scheduling](https://w3c.github.io/requestidlecallback) and it takes [inspiration](https://en.wikipedia.org/wiki/Cooperative_multitasking) from the operative systems world.

So thanks to cooperative scheduling `Fiber` is able to interrupt work when the main thread need to take care of something more urgent, and this for the triangles demo reflects on a smooth animation without dropping frames.

## When React Fiber will be released ?
Really soon in the next major version and **without changing the public API**.

Facebook folks care a lot about [stability](https://facebook.github.io/react/contributing/design-principles.html#stability). They do not like breaking changes because they have something like 30.000 components to maintain.

> `Fiber` is expected to be enabled by default in **React 16**.

The first releases will focus on **backward compatibility** and **not** on new features.

Facebook is already using `React Fiber` in its beta version. You can try it out [here](https://www.beta.facebook.com).

You can play with `Fiber` today simply typing:

```
npm install react@next react-dom@next
```

## What new features will React Fiber introduce ?
`React Fiber` opens the door to a lot of new functionalities. These are the ones the react core team has already started working on.

* [Defer updates to avoid dropping frames (Async setState / Time slicing)](https://github.com/facebook/react/issues/8830)
* [Recover from errors thrown in render methods (Error Boundaries)](https://github.com/facebook/react/issues/2461)
* [Render subtrees into DOM node containers (Portals)](https://github.com/facebook/react/pull/8386)
* [Render can return multiple children](https://github.com/facebook/react/issues/2127)

<small>NB: These new features will not be available on React 16 which, as I already said, will mainly focus on backward compatibility.</small>

We already had a taste of how the new `unstable_deferredUpdates` API works on the triangles demo.

I am planning to analyse the rest of the new experimental functionalities as I have some free time in different posts. For now you can click on the links above to have an overview of what they are introducing.

## Conclusion
So in a nutshell why should you be excited about this shiny new version of the React reconciler algorithm?

* It makes apps more fluid and responsible allowing high priority updates to jump ahead low priority updates. It does that breaking up the work into small unit of works that can be paused.
* In future it could parallelise work between multiple workers in an efficient way splitting up branches of the tree and analyse them in parallel.
* In the long term it would improve startup time rendering components as they become available to the browser without the need to wait for a whole bundle to be downloaded.

> **The future looks bright and full of new possibilities for React**

### References
* [A cartoon intro to Fiber](https://www.youtube.com/watch?list=PLb0IAmt7-GS3fZ46IGFirdqKTIxlws7e0&v=ZCuYPiUIONs)
* [Unofficial React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)
* [W3C: Cooperative Scheduling of Background Tasks](https://www.w3.org/TR/requestidlecallback)
* [Fiber Principles: Contributing To Fiber](https://github.com/facebook/react/issues/7942)
* [Is Fiber Ready Yet?](http://isfiberreadyyet.com)
