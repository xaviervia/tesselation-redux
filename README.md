# Tesselation (thesis)

This project is a thesis on how to build front end applications.

**Tesselation** is a simple but not trivial application with various features that require several distinct side effects, such as:

- Auto saving locally in the browser.
- Automatic synchronization across browser tabs/windows.
- Logging updates to the console.
- Rendering an SVG diagram and doing mouse interaction with it.
- Seeding data from random.

The purpose of having several distinct side effect types is to explore a common way of dealing with them. The side effects are chosen because they are realistic requirements of many modern front end applications, but they are also stand ins for any side effect sharing their characteristics – the [effect directionality](#effect-directionality). More on that later.

Another purpose of the thesis is the apply to the architecture several principles that are very present in the current front end programming trends:

- Side effects are wired to the core logic with a reactive programming approach.
- Core application logic is purely functional: that is, it's composed entirely of pure functions, and it's implemented using functional programming patterns.
- Core application logic is also composable, and this is done by composing high order reducers.
- All side effects are treated in the same way, and divided into three subcategories:
  - Outgoing
  - Incoming
  - Bidirectional

Also, as much as possible, the implementation does not rely on libraries that would obscure it and make the thesis of how to build the app dependent on the particular choice of tools. So you could say that minimalism is also one of the goals.

And before going on, a disclaimer: please don't take this project too seriously or assume that I'm completely sold on the ideas that I put together here. This is an experiment, and while I'm rather happy with the results, there are no simple answers in programming.

## Let's get started

First, open the live app in [https://xaviervia.github.io/tesselation/](https://xaviervia.github.io/tesselation/). You will something similar to:

![tesselation screenshot](images/tesselation-app.png)

> This app was tested and intended for a recent version of Chrome. It might not work somewhere else.

What you are seeing is a very simple example of a [Voronoi tesselation](https://en.wikipedia.org/wiki/Voronoi_diagram), a kind of cell diagram generated from points in a plane. This app uses 9 points for the tesselation to keep things concise. If this is the first time you open the application, the distribution of dots that you will see is one generated by the [Seed](#seed) effect, which created a list with 9 points in with random values of x and y between 0 and 99.

Let's try a couple of things to demonstrate the features of the app:

1. Reload the application: you will see that the distribution of dots didn't change. This is because the distribution of dots was actually saved by the [LocalStorage](#localStorage) to, well, `localStorage`, and recovered from there automatically when restarting.

2. Resize the window: you will see that the tesselation changes it's proportions to fit the new window size. The way this works is that the [Resize](#resize) effect notifies each time the window got resized, and the positions of the dots and lines are recalculated using the new size to schew the 100x100 grid.

3. Click around: one dot will be removed, then another point will be aded in roughly the position where you clicked (roughly because positions are normalized from whatever size your window is to a 100x100 grid). This action is sent from the [View](#view) – which is React.

4. Open the console and click around in the app: you will see that each new distribution of dots is logged to the console (the [Log](#log) effect is responsible for this). Try clicking several times in the same place without moving the mouse: you will see that the distribution of dots is printed only once. The state management part of the application is taking care of not sending the same state twice to the effects. We will see this later in more detail.

5. Now that you clicked around a couple of times, click the `⎌` (undo) button: you will see that the point distribution goes back to the previous one. How this is achieved is an interesting part of the state manipulation and [state logic composition](#state-logic-composition) exploration.

6. Try clearing up the whole thing (?) it will be reseeded.

## Core application logic

The whole application state is contained in a single object blob, Redux-style. Actually, the whole architecture is heavily inspired by Redux, but with two core differences:

- Not using Redux, to prove that the "single object" state approach goes beyond libraries and it's easily reproducible with any reactive programming library.
- Higher decoupling from the UI: Redux is originally meant to represent the data necessary to drive the UI, and the patterns that emerged around it reflect that intent. The thesis here is that the way Redux drives state actually applies to the management of any type of side effect, not just UI, and for that purpose I introduced a generalized wiring interface that, the thesis goes, can be used for _any_ side effect.

To emphasize the fact that the architecture is meant to be a generalization of Redux and not another Flux implementation (?), the nomenclature is slightly different:
- `dispatch` is called `push` to represent the fact that the actual operation of dispatching an action is analogous to pushing into an array structure. As a matter of fact, it's pushing data into a stream that get's reduced on each addition – or `scan`ned in `flyd` lingo.
- There is no `getState`. State is always pushed to the effects as an object each time the application state is updated.

## Effects

Here comes another semantic part of the thesis: I'm renaming _side effects_ as _effects_ since, as many pointed out, an application without side effects is just a way of transforming electricity into heat. _Effects_ is a correct name: the purpose of the application are in fact it's effects, while side effects refer to the _unintended_ effects of performing a purely functional operation.

In the `src/effects` folder you can find the entry points (and in the case of this application, being so simple, the whole implementation) of all the side effects of the application. They can be cataloged into three categories, and although this categories are not extremely useful they are helpful to understand the why of the wiring API, so I'll explain them briefly:

### Effect directionality

- Incoming: Effects that only inject data into the application.
  - [Resize](src/effects/resize.js) listens to `resize` events on the window and pushes an action with the new value.
  - [Setup](src/effects/setup.js) when initialized, it immediately pushes an action with a newly generated UUID to identify the instance of the application. There is an interesting gotcha here: I initially modeled this as being part of the `initialState` object in the `store`, but you can see how that violates the purity of the store implementation. It's a common temptation to include "one off" effects in the creation of the `initialState`, but that is likely to create problems down the line. It's a good litmus test for the store implementation that no libraries with side effects are used in it.

- Outgoing: Effects that react to the new state by performing some operation, but never inject anything back.
  - [Log](src/effects/log.js) which simply logs the current points each time the state is updated.

- Bidirectional: Effects that react to the state _and_ inject information into it via actions:
  - [LocalStorage](src/effects/localStorage.js): when first invoked, it checks if there is an entry in `localStorage` for `tesselation`, and if there is, it immediately pushes an action with the previously saved state. It also sets up a listener on the window `storage` event, and whenever said `tesselation` entry is updated it pushes another state override, thus keeping it in sync with whatever other instance of the application is running in a different window or tab (this will conspire with another effect and come back to bite us in one of the gotchas).

### Seed



## Debugging


## Gotchas and easter eggs

The application has two bugs that were left there to demonstrate the kind of quirks that can emerge out of this architecture, and I will discuss here possible solutions.

TODO: Where should the logic for preprocessing the actions be done?

## Take aways

- **Symbols** make for pretty cool action types.

## Credits and references

- [Redux](http://redux.js.org/) which heavily inspired this architecture.
- @joaomilho who originally gave me the idea of using [`flyd`](https://github.com/paldepind/flyd) to re implement the Redux store.
- @Nevon's [demystifying Redux](https://gist.github.com/Nevon/eada09788b10b6a1a02949ec486dc3ce)
-
