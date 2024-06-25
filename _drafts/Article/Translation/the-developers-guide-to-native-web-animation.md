---
title: The developer’s guide to native web animation | Heart Internet Blog –
  Focusing on all aspects of the web
authorURL: ""
originalURL: https://www.heartinternet.uk/blog/the-developers-guide-to-native-web-animation/
translator: ""
reviewer: ""
---

08/01/2019

<!-- more -->

Posted in [Guest Posts][1], [Web Design][2]

Animation on the web has come a long way. At first we used Flash to create websites, which were all fun and interactive, then we got to fancy JavaScript libraries, and now the web platform offers us a selection of native animation tools. But with all the new choices it gets harder to decide which ones to use, so we’re going to dive deeper into native animation options currently available and what the differences are.

### CSS animations and transitions

The simplest and most popular option is CSS animation and transitions. With its declarative nature CSS offers a basic way to add some animations to elements directly in your CSS.

CSS transitions are a good use case for animation, when you want to go from one state A to another state B. Here is a little example of a menu animation:

See the Pen [Menu Left][3] by Lisi ([@lisilinhart][4]) on [CodePen][5].

Whenever the menu on the left is hovered, its transform property is set to `transform: translate(0%);`. When it’s not hovered, it’s set to `transform: translateX(-90%);`. Then on the idle not-hovered state we’re setting a transition on our transform property `transition: transform 0.45s cubic-bezier(0.39, 0.575, 0.565, 1);`. So our menu is going from state _90% to the left_ to _0% in the center_.

CSS animations on the other hand allow us to create animations with more complex animation states. So if you want to animate from State A to State B to State C to State D, etc. These kind of animations are perfect for simple loading animations. Here’s another little example:

See the Pen [Loading][6] by Lisi ([@lisilinhart][7]) on [CodePen][8].

On our element that we want to be animated we define the animation to load `animation: pulse both 1.2s infinite linear;`. This tells the animation to run with a speed of 1.2s, infinitely, a linear easing function and an fill-mode of both. If you’re not familiar with the syntax, take a look at [Animations][9] [in CSS][10] on [cssreference.io][11] and explore what all the keywords mean. The actual definition of our animation happens with the `@keyframes` pulse definition. Essentially, we have three states in our animation:

1.  Scaled to almost zero and not `visible opacity: 0`. With the ease-out-sine timing function `animation-timing-function: cubic-bezier(0.39, 0.575, 0.565, 1`) we’re telling the animation to come in slowly and then speed up until we reach the next state.
2.  Scaled to its actual size and visible with a timing function of linear to make it slower in the middle of the animation.
3.  Scaled up to 1.4 times its size and fading to not visible with an ease-in-sine function to make it disappear slowly and then speeding up to the end of the animation.

```
@keyframes pulse {
  0% {
    transform: rotate(45deg) scale3d(0.1, 0.1, 0.1);
      opacity: 0;
      animation-timing-function: cubic-bezier(0.39, 0.575, 0.565, 1);
  }

  50% {
    transform: rotate(45deg) scale3d(1, 1, 1);
      opacity: 1;
    animation-timing-function: linear;
  }

  100% {
    transform: rotate(45deg) scale3d(1.4, 1.4, 1.4);
    opacity: 0;
      animation-timing-function: cubic-bezier(0.47, 0, 0.745, 0.715);
  }
}
```

A great bonus for CSS transitions and animations are that they can run independently from your JavaScript and are able to be hardware accelerated over your GPU. Good use cases for both are: Simple animations with up to three different stages, loading spinners, and animations triggered on `:hover` and `:focus`. However, if you want to create more complex animations that have lots of different stages, this is fairly hard to do in CSS, since you would need to write a lot of CSS code that’s perfectly timed together.

### The Web Animations API

This is where the Web Animations API (short WAAPI) comes in, a more powerful, native alternative to CSS animation. Similar to CSS animations the WAAPI can run animations on the compositor thread, so independently from your JavaScript, and it can also hardware accelerate animations over the GPU. Furthermore, it gives you controls and callback to handle your animations after they were created. Let’s take a look at an example:

See the Pen [Waapi][12] by Lisi ([@lisilinhart][13]) on [CodePen][14].

We have a counter that’s counting down from 10, and we want to speed up the animation the closer the countdown gets to 0. To create animations with the Web Animations API, we first have to get an element we want to animate with `document.querySelector`, then we need to call the `.animate()` function on this element. This returns an animation object with certain controls and callbacks, which we’re going to use later. To create an animation, the .animate function takes the following parameters:

1.  A keyframes array or object with the different stages of the animation.
2.  A timings object with the different timing options on how fast and how often the animation should run.

In the example below we have two keyframes: from visible and scaled to 60% to half visible and scaled to its full size. For the timing we defined it to run half a second with a linear easing, no delay and 100 iterations. Again these keywords are almost the same keywords we use in CSS animations.

```
const clock = document.querySelector('.countdown');
const countdownAnimation = clock.animate([
  {opacity: 1, transform: 'scale(.6)'},
  {opacity: .5, transform: 'scale(1)'}
], {
  duration: 500,
  easing: 'linear',
  delay: 0,
  iterations: 100,
  direction: 'alternate',
  fill: 'forwards'
});
```

To speed up the animation, we make use of the control on our `countdownAnimation` object, which is returned by the `.animate` function in the code snippet above. We’re setting an interval that’s counting down our numbers, and as long as our number is bigger than 0, we’re increasing our `countdownAnimation.playbackRate` up to a maximum of 6. If our counter is finished, we’re calling the `countdownAnimation.finish()` control function to stop our animation and clear the interval.

```
const interval = setInterval(function() {
  time = time - 1;
  clock.textContent = time;
  if (time > 0) {
    countdownAnimation.playbackRate = Math.min(countdownAnimation.playbackRate * 1.15, 6);
  } else {
    countdownAnimation.finish();
        clearInterval(interval);
  }
}, 1000);
```

Since we’re defining our animations in JavaScript, we get a lot more control and can create more complex and interactive animations. If we had done the following example in CSS, we would have needed multiple elements, which would have needed to be timed perfectly together. Doing it with the WAAPI makes the code a lot more readable. The only downside to the WAAPI at the moment is the missing full browser support . If we take a look at the [Can I][15] [Use][16] numbers, Firefox, Chrome and Safari are getting good support, but Edge hasn’t yet implemented the API, so it’s going to take a little bit longer until your animations run everywhere. The general advice for browser support with the Web Animations API is to fall back to no animations if the browser doesn’t support them. There is a [polyfill][17], if you really need full browser support, but not everyone recommends using it.

### JavaScript and CSS Variables

The third popular option for web animation are CSS Variables in combination with JavaScript. Many developers love this approach to animation, because it allows you to define your complex calculations in JavaScript. You then specify which element gets animated and which of its CSS properties is changed in CSS. JavaScript in combination with CSS Variables is especially great for creating reactive animations.

Since CSS Variables are inherited by their parent, this allows us to define a variable on a parent element and make use of this variable in all child elements. This is great because we don’t need excessive DOM manipulation and only define the variable on the parent once.

Let’s have a look at an example:

See the Pen [Javascript & CSS Variables][18] by Lisi ([@lisilinhart][19]) on [CodePen][20].

First we need to get the element we want to set our variable on. This will be the parent element, with all the child elements that we want to animate inside. We’re defining a `raf` variable to be able to call a function for continuous animation optimisation called `requestAnimationFrame`.

```
const parent = document.querySelector('#app');
let raf; 
```

Then we’re adding a `mousemove` listener on our document. So whenever the mouse is moved in our document, a new event is fired. When that event is fired, we’re calling our update function with the `requestAnimationFrame` function that’s provided by the browser. Essentially, this tells the browser that we’re updating an animation and that the provided function should be called before the browser performs the next repaint. The `requestAnimationFrame` is widely used for improved performance on continuous animation updates.

```
document.addEventListener('mousemove', (e) => {
  raf = raf || requestAnimationFrame(() => update(e));
});
```

Then we’re defining our actual update function. Here we’re calculating our `e.clientX` pixel value in proportion to the window width. The calculation returns an `x` value of `-1` when the mouse is on the left hand side of the document, a value of `0` when it’s in the centre and a value of `1`, when it’s on the right.

Finally we’re setting our variable on our parent element with the `setProperty('--x', x )` function. In the end we’re setting our `raf` function to null, to be able to call `requestAnimationFrame` again.

```
function update(e){
    const x =  ((e.clientX / window.innerWidth) - 0.5) * 2;
  parent.style.setProperty('--x', x );
  raf = null;
}
```

This is all the JavaScript we need for our reactive animations, now let’s have a look at the CSS. To be able to animate one of the child elements, we’re going to change their transform and opacity properties. For the first item we’re changing the rotation according to mouse position. The mouse value is already set on the parent `#app` element via JavaScript and accessible via CSS over the `var(—x)` selector. Since our X Value is between -1 and 1, depending on where the mouse currently is at, we need to multiply this value with degrees. So with `rotate(calc(var(--x) * 360deg))` we’re rotating our element 360 degrees clockwise or anticlockwise depending on if you’re moving your mouse left or right. Similar calculations are made with the other elements. Since we set X to a sensible value, it’s easy to multiply it with different units like `px` or `deg` or `%`, whatever you need for your transformation.

```
.item-one {
        transform: rotate(calc(var(--x) * 360deg));
}
```

Using CSS Variables this way allows us to avoid excessive DOM Manipulation, because we only need to set the variable on the parent once, thus making it accessible to all child elements.

They also allow us to animate different transform properties independently. So for example rotating an element independently from its translation, by animating a rotation variable. Using JavaScript to do calculations allows us to use more complex easings, like a bouncing easing or react to DOM events. Generally, CSS Variables and JavaScript go really well together.

### JavaScript animations

I’m not going to dive deep into JavaScript animation, because there are a lot of JavaScript libraries for all the different animation needs.

Have a look at [Web-animation toolkit][21] and choose the right library for what you want to animate. Generally, libraries are a great choice if you want to animate SVG, since there are still a lot of browser inconsistencies if you do more complex SVG animation natively. There are also amazing libraries like [three.js][22] for 3D WebGL animation or other 2D renderers like [PixiJS][23].

In general I’d recommended first trying to solve simpler animations with CSS or the Web Animations API, before going for a JavaScript library.

However, if you need to create very animation-heavy websites, animation tools like [GSAP][24] can be very useful, since you get amazing, performant, cross-browser consistent results in a short amount of time. It’s not a tiny library, though, so consider beforehand if the code overhead is worth adding to your project.

By Lisi Linhart  
in [Guest Posts][25], [Web Design][26]  
on 08/01/2019

 [share][27] [Tweet][28]

[1]: https://www.heartinternet.uk/blog/category/guest-posts/
[2]: https://www.heartinternet.uk/blog/category/web-design/
[3]: https://codepen.io/lisilinhart/pen/bQvVmo/
[4]: https://codepen.io/lisilinhart
[5]: https://codepen.io
[6]: https://codepen.io/lisilinhart/pen/xQWwox/
[7]: https://codepen.io/lisilinhart
[8]: https://codepen.io
[9]: https://cssreference.io/animations/
[10]: https://cssreference.io/animations/
[11]: https://cssreference.io/
[12]: https://codepen.io/lisilinhart/pen/JeLGqN/
[13]: https://codepen.io/lisilinhart
[14]: https://codepen.io
[15]: https://caniuse.com/#feat=web-animation
[16]: https://caniuse.com/#feat=web-animation
[17]: https://github.com/web-animations/web-animations-js
[18]: https://codepen.io/lisilinhart/pen/pQLyOR/
[19]: https://codepen.io/lisilinhart
[20]: https://codepen.io
[21]: https://web-animation.github.io/
[22]: https://threejs.org/
[23]: http://www.pixijs.com/
[24]: https://greensock.com/gsap
[25]: https://www.heartinternet.uk/blog/category/guest-posts/
[26]: https://www.heartinternet.uk/blog/category/web-design/
[27]: https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.heartinternet.uk%2Fblog%2Fthe-developers-guide-to-native-web-animation%2F
[28]: https://twitter.com/intent/tweet?text=The+developer%E2%80%99s+guide+to+native+web+animation&url=https%3A%2F%2Fwww.heartinternet.uk%2Fblog%2Fthe-developers-guide-to-native-web-animation%2F&via=heartinternet