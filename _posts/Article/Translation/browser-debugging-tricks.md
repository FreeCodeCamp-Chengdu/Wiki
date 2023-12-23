---
title: 67 Weird Debugging Tricks Your Browser Doesn't Want You to Know
authorURL: ""
originalURL: https://alan.norbauer.com/articles/browser-debugging-tricks
translator: ""
reviewer: ""
---

A list of useful, not-obvious hacks to get the most out of your browser’s[1](#user-content-fn-1) debugger. Assumes an intermediate-level-or-higher understanding of the developer tools.

## [Advanced Conditional Breakpoints](#advanced-conditional-breakpoints)

By using expressions that have side effects in places you wouldn’t expect, we can squeeze more functionality out of basic features like conditional breakpoints.

### [Logpoints / Tracepoints](#logpoints--tracepoints)

For example, we can `console.log` in breakpoints. Logpoints are breakpoints that log to the console without pausing execution. While Microsoft Edge has had logpoints built-in for a while and Chrome just added them in v73, Firefox does not. But, we can use conditional breakpoints to simulate them in any browser.

![Conditional Breakpoint - console.log](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-console-log.2d18d3e4.gif&w=1920&q=75)

Use `console.count` instead of `console.log` if you also want a running count of how many times the line is executed.

UPDATE (May 2020): All the major browsers now directly support logpoints/tracepoints ([Chrome Logpoints](https://developers.google.com/web/updates/2019/01/devtools#logpoints), [Edge Tracepoints](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger#breakpoints), [Firefox Logpoints](https://developer.mozilla.org/en-US/docs/Tools/Debugger/Set_a_logpoint))

#### [Watch Pane](#watch-pane)

You can also use `console.log` in the watch pane. For example, to dump a snapshot of `localStorage` everytime your application pauses in the debugger, you can create a `console.table(localStorage)` watch:

![console.table in watch pane](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-table-in-watch.03919d55.png&w=1080&q=75)

Or to execute an expression after DOM mutation, set a DOM mutation breakpoint (in the Element Inspector): ![DOM Mutation Breakpoint](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-DOM-mutation-chrome.27f07619.png&w=1920&q=75)

And then add your watch expression, e.g. to record a snapshot of the DOM: `(window.doms = window.doms || []).push(document.documentElement.outerHTML)`. Now, after any DOM subtree modification, the debugger will pause execution and the new DOM snapshot will be at the end of the `window.doms` array. (There is no way to create a DOM mutation breakpoint that doesn’t pause execution.)

#### [Tracing Callstacks](#tracing-callstacks)

Let’s say you have a function that shows a loading spinner and a function that hides it, but somewhere in your code you’re calling the show method without a matching hide call. How can you find the source of the unpaired show call? Use `console.trace` in a conditional breakpoint in the show method, run your code, find the last stack trace for the show method and click the caller to go to the code:

![console.trace in conditional breakpoint](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-trace-find-stack.d107e89c.gif&w=1920&q=75)

### [Changing Program Behavior](#changing-program-behavior)

By using expressions that have side effects on program behavior, we can change program behavior on the fly, right in the browser.

For example, you can override the param to the `getPerson` function, `id`. Since `id=1` evaluates to true, this conditional breakpoint would pause the debugger. To prevent that, append `, false` to the expression.

![Conditional Breakpoint - parameter override](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-parameter-override.375af5d5.gif&w=1920&q=75)

### [Quick and Dirty Performance Profiling](#quick-and-dirty-performance-profiling)

You shouldn’t muddy your performance profiling with things like conditional breakpoint evaluation time, but if you want a quick and dirty measurement of how long something takes to run, you can use the console timing API in conditional breakpoints. In your starting point set a breakpoint with the condition `console.time('label')` and at the end point set a breakpoint with the condition `console.timeEnd('label')`. Everytime the thing you’re measuring runs, the browser will log to the console how long it takes.

![Conditional Breakpoint - performance profile](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconsole-time-performance-profile.9b494665.gif&w=1920&q=75)

### [Using Function Arity](#using-function-arity)

#### [Break on Number of Arguments](#break-on-number-of-arguments)

Only pause when the current function is called with 3 arguments: `arguments.callee.length === 3`

Useful when you have an overloaded function that has optional parameters.

![Conditional Breakpoint - argument length](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-argument-length.eedb2e1c.gif&w=1920&q=75)

#### [Break on Function Arity Mismatch](#break-on-function-arity-mismatch)

Only pause when the current function is called with the wrong number of arguments: `(arguments.callee.length) != arguments.length`

![Conditional Breakpoint - arity check](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-arity-check.70c0a60c.gif&w=1920&q=75)

Useful when finding bugs in function call sites.

### [Using Time](#using-time)

#### [Skip Page Load](#skip-page-load)

Don’t pause until 5 seconds after page load: `performance.now() > 5000`

Useful when you want to set a breakpoint but you’re only interested in pausing execution after initial page load.

#### [Skip N Seconds](#skip-n-seconds)

Don’t pause execution if the breakpoint is hit in the next 5 seconds, but pause anytime after: `window.baseline = window.baseline || Date.now(), (Date.now() - window.baseline) > 5000`

Reset the counter from the console anytime you’d like: `window.baseline = Date.now()`

### [Using CSS](#using-css)

Pause based on computed CSS values, e.g. only pause execution when the document body has a red background color: `window.getComputedStyle(document.body).backgroundColor === "rgb(255,0,0)"`

### [Even Calls Only](#even-calls-only)

Only pause every other time the line is executed: `window.counter = window.counter || 0, window.counter % 2 === 0`

### [Break on Sample](#break-on-sample)

Only break on a random sample of executions of the line, e.g. only break 1 out of every 10 times the line is executed: `Math.random() < 0.1`

### [Never Pause Here](#never-pause-here)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)

When you right-click the gutter and select “Never Pause Here,” Chrome creates a conditional breakpoint that is `false` and never passes. This makes it so that the debugger will never pause on this line.

![Never Pause Here](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fnever-pause-here.a4010ee4.png&w=640&q=75) ![Never Pause Here Result](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fnever-pause-here-result.32dc71c3.png&w=750&q=75)

Useful when you want to exempt a line from XHR breakpoints, ignore an exception that is being thrown, etc.

### [Automatic Instance IDs](#automatic-instance-ids)

Automatically assign a unique ID to every instance of a class by setting this conditional breakpoint in the constructor: `(window.instances = window.instances || []).push(this)`

Then to retrieve the unique ID: `window.instances.indexOf(instance)` (e.g. `window.instances.indexOf(this)` when in a class method)

### [Programmatically Toggle](#programmatically-toggle)

Use a global boolean to gate one or more conditional breakpoints:

![Boolean gate](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-gated.d32764ce.png&w=1920&q=75)

Then programmatically toggle the boolean, e.g.

-   manually, from the console
    
    \[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }
    
    ```javascript
    1window.enableBreakpoints = true;
    ```
    
    \[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }
    
    ```javascript
    1window.enableBreakpoints = true;
    ```
    
-   from other breakpoints ![Boolean gate - enable from other breakpoint](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-gated-enable-from-breakpoint.1c568b6e.png&w=1920&q=75)
-   from a timer on the console
    
    \[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }
    
    ```javascript
    1setTimeout(() => (window.enableBreakpoints = true), 5000);
    ```
    
    \[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }
    
    ```javascript
    1setTimeout(() => (window.enableBreakpoints = true), 5000);
    ```
    
-   etc

## [monitor() class Calls](#monitor-class-calls)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)

You can use Chrome’s `monitor` command line method to easily trace all calls to class methods. E.g. given a class `Dog`

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1class Dog {2  bark(count) {3    /* ... */4  }5}
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1class Dog {2  bark(count) {3    /* ... */4  }5}
```

If we want to know all calls made to all instances of `Dog`, paste this into the command line:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1var p = Dog.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1var p = Dog.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

and you’ll get output in the console:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) }

```
> function bark called with arguments: 2
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) }

```
> function bark called with arguments: 2
```

You can use `debug` instead of `monitor` if you want to pause execution on any method calls (instead of just logging to the console).

### [From a Specific Instance](#from-a-specific-instance)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)

If you don’t know the class but you have an instance:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1var p = instance.constructor.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1var p = instance.constructor.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
```

Useful when you’d like to write a function that does this for any instance of any class (instead of just `Dog`)

## [Call and Debug a Function](#call-and-debug-a-function)

Before calling the function you want to debug in the console, call `debugger`. E.g. given:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1function fn() {2  /* ... */3}
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1function fn() {2  /* ... */3}
```

From your console:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) }

```
> debugger; fn(1);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) }

```
> debugger; fn(1);
```

And then “Step into next function call” to debug the implementation of `fn`.

Useful when you don’t feel like finding the definition of `fn` and adding a breakpoint manually or if `fn` is dynamically bound to a function and you don’t know where the source is.

In Chrome you can also optionally call `debug(fn)` on the command line and the debugger will pause execution inside `fn` every time it is called.

## [Pause Execution on URL Change](#pause-execution-on-url-change)

To pause execution before a single-page application modifies the URL (i.e. some routing event happens):

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1const dbg = () => {2  debugger;3};4history.pushState = dbg;5history.replaceState = dbg;6window.onhashchange = dbg;7window.onpopstate = dbg;
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1const dbg = () => {2  debugger;3};4history.pushState = dbg;5history.replaceState = dbg;6window.onhashchange = dbg;7window.onpopstate = dbg;
```

Creating a version of `dbg` that pauses execution without breaking navigation is an exercise left up to the reader.

Also, note that this doesn’t handle when code calls `window.location.replace/assign` directly because the page will immediately unload after the assignment, so there is nothing to debug. If you still want to see the source of these redirects (and debug your state at the time of redirect), in Chrome you can `debug` the relevant methods:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1debug(window.location.replace);2debug(window.location.assign);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1debug(window.location.replace);2debug(window.location.assign);
```

## [Debugging Property Reads](#debugging-property-reads)

If you have an object and want to know whenever a property is read on it, use an object getter with a `debugger` call. For example, convert `{configOption: true}` to `{get configOption() { debugger; return true; }}` (either in the original source code or using a conditional breakpoint).

Useful when you’re passing in some configuration options to something and you’d like to see how they get used.

## [Use copy()](#use-copy)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)![Firefox](/_next/static/media/firefox.583d9a58.svg)

You can copy interesting information out of the browser directly to your clipboard without any string truncation using the `copy()` console API. Some interesting things you might want to copy:

-   Snapshot of the current DOM: `copy(document.documentElement.outerHTML)`
-   Metadata about resources (e.g. images): `copy(performance.getEntriesByType("resource"))`
-   A large JSON blob, formatted: `copy(JSON.parse(blob))`
-   A dump of your localStorage: `copy(localStorage)`
-   Etc.

## [Debugging HTML/CSS](#debugging-htmlcss)

The JS console can be helpful when diagnosing problems with your HTML/CSS.

### [Inspect the DOM with JS Disabled](#inspect-the-dom-with-js-disabled)

When in the DOM inspector press ctrl+\\ (Chrome/Windows) to pause JS execution at any time. This allows you to inspect a snapshot of the DOM without worrying about JS mutating the DOM or events (e.g. mouseover) causing the DOM to change from underneath you.

### [Inspect an Elusive Element](#inspect-an-elusive-element)

Let’s say you want to inspect a DOM element that only conditionally appears. Inspecting said element requires moving your mouse to it, but when you try to, it disappears:

![Elusive element](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Felusive-element.495b0945.gif&w=1920&q=75)

To inspect the element you can paste this into your console: `setTimeout(function() { debugger; }, 5000);`. This gives you 5 seconds to trigger the UI, and then once the 5 second timer is up, JS execution will pause and nothing will make your element disappear. You are free to move your mouse to the dev tools without losing the element:

![Elusive element - inspected](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Felusive-element-inspected.f5f036b4.gif&w=1920&q=75)

While JS execution is paused you can inspect the element, edit its CSS, execute commands in the JS console, etc.

Useful when inspecting DOM that is dependent on specific cursor position, focus, etc.

### [Record Snapshots of the DOM](#record-snapshots-of-the-dom)

To grab a copy of the DOM in its current state:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1copy(document.documentElement.outerHTML);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1copy(document.documentElement.outerHTML);
```

To record a snapshot of the DOM every second:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1doms = [];2setInterval(() => {3  const domStr = document.documentElement.outerHTML;4  doms.push(domStr);5}, 1000);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1doms = [];2setInterval(() => {3  const domStr = document.documentElement.outerHTML;4  doms.push(domStr);5}, 1000);
```

Or just dump it to the console:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1setInterval(() => {2  const domStr = document.documentElement.outerHTML;3  console.log("snapshotting DOM: ", domStr);4}, 1000);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1setInterval(() => {2  const domStr = document.documentElement.outerHTML;3  console.log("snapshotting DOM: ", domStr);4}, 1000);
```

### [Monitor Focused Element](#monitor-focused-element)

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1(function () {2  let last = document.activeElement;3  setInterval(() => {4    if (document.activeElement !== last) {5      last = document.activeElement;6      console.log("Focus changed to: ", last);7    }8  }, 100);9})();
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1(function () {2  let last = document.activeElement;3  setInterval(() => {4    if (document.activeElement !== last) {5      last = document.activeElement;6      console.log("Focus changed to: ", last);7    }8  }, 100);9})();
```

![Monitor focused element](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fmonitor-focus.b9692b99.gif&w=1920&q=75)

### [Find Bold Elements](#find-bold-elements)

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1const isBold = (e) => {2  let w = window.getComputedStyle(e).fontWeight;3  return w === "bold" || w === "700";4};5Array.from(document.querySelectorAll("*")).filter(isBold);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1const isBold = (e) => {2  let w = window.getComputedStyle(e).fontWeight;3  return w === "bold" || w === "700";4};5Array.from(document.querySelectorAll("*")).filter(isBold);
```

#### [Just Descendants](#just-descendants)

Or just descendants of the element currently selected in the inspector:

\[data-bright-mode="dark"\] { display: block } :root.light \[data-bright-mode="dark"\] { display: none } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1Array.from($0.querySelectorAll("*")).filter(isBold);
```

\[data-bright-mode="light"\] { display: none } :root.light \[data-bright-mode="light"\] { display: block } \[data-bright-theme\] ::selection { background-color: var(--selection-background) } \[data-bright-theme\] \[data-bright-ln\] { color: var(--line-number-color); margin-right: 1.5ch; display: inline-block; text-align: right; user-select: none; }

```javascript
1Array.from($0.querySelectorAll("*")).filter(isBold);
```

### [Reference Currently Selected Element](#reference-currently-selected-element)

`$0` in the console is an automatic reference to the currently selected element in the element inspector.

#### [Previous Elements](#previous-elements)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)![Edge](/_next/static/media/edge.c22c90ce.svg)

In Chrome and Edge you can access the element you last inspected with `$1`, the element before that with `$2`, etc.

#### [Get Event Listeners](#get-event-listeners)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)

In Chrome you can inspect the event listeners of the currently selected element: `getEventListeners($0)`, e.g.

![getEventListeners](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2FgetEventListeners.4ae6f43e.png&w=1920&q=75)

### [Monitor Events for Element](#monitor-events-for-element)

![Chrome](/_next/static/media/chrome.2d2a19fd.svg)

Debug all events for selected element: `monitorEvents($0)`

Debug specific events for selected element: `monitorEvents($0, ["control", "key"])`

![monitorEvents](/_next/image?url=%2F_next%2Fstatic%2Fmedia%2FmonitorEvents.a03f9e53.gif&w=1920&q=75)

## [Footnotes](#footnote-label)

1.  Tips are supported in Chrome, Firefox, and Edge unless the browser logos say otherwise: ![Chrome](/_next/static/media/chrome.2d2a19fd.svg) ![Firefox](/_next/static/media/firefox.583d9a58.svg) ![Edge](/_next/static/media/edge.c22c90ce.svg) [↩](#user-content-fnref-1)