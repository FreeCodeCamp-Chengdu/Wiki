---
title: A Brief Introduction to Web Components
date: 2025-05-08T15:32:15.694Z
author: Mark Mahoney
authorURL: https://www.freecodecamp.org/news/author/markm208/
originalURL: https://www.freecodecamp.org/news/a-brief-introduction-to-web-components/
translator: ""
reviewer: ""
---

/ [#Web Components][1]

<!-- more -->

# A Brief Introduction to Web Components

![Mark Mahoney](https://cdn.hashnode.com/res/hashnode/image/upload/v1742697935887/aa3553a0-8522-481b-bda8-5bf4e1df7249.png)

[Mark Mahoney][2]

  ![A Brief Introduction to Web Components](https://cdn.hashnode.com/res/hashnode/image/upload/v1746625674217/902b1ac1-f7c2-42cd-b12f-d9803e58739d.png)

In a previous article, I gave a [brief introduction to React][3]. This tutorial introduces an alternative approach to building a component-based frontend. It covers the fundamentals of [Web Components][4] to build modular, reusable elements for your web applications.

Web Components are a set of standardized browser APIs that allow you to create custom, reusable HTML elements with encapsulated functionality. They help developers create self-contained components that can be used across different frameworks or even without any framework at all.

This tutorial assumes you have some basic programming experience and are comfortable reading and writing JavaScript. You should understand variables, functions, loops, objects, classes, and how JavaScript works in the browser. You don’t need to know anything about Web Components to get started.

The four lessons presented here are taken from my free book of code playbacks:

> [An Introduction to Web Development from Back to Front][5]  
> By Mark Mahoney

This book is available for free on [Playback Press][6]. The book is a hands-on guide to modern web development, covering everything from core JavaScript features to building full-stack apps with various tools and technologies.

Each lesson is presented as a [code playback][7], which is an interactive code walkthrough that shows how a program changes over time along with my explanation about what's happening. This format helps you focus on the reasoning behind the code changes.

To view a playback, click on the comments in the left panel. Each comment updates the code in the editor and highlights the change. Read the explanation, study the code, and use the built-in AI tutor if you have questions. Here's a short video that shows how to use a code playback:

<iframe width="560" height="315" src="https://www.youtube.com/embed/uYbHqCNjVDM" style="aspect-ratio: 16 / 9; width: 100%; height: auto;" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen="" loading="lazy"></iframe>

After this introduction, you might want to explore the official Web Components resources: [MDN Web Components Guide][8].

## **Table of Contents**

-   [Web Components Part 1: Your First Custom Element][9]
    
-   [Web Components Part 2: Data Communication][10]
    
-   [Web Components Part 3: Custom Events][11]
    
-   [Web Components Part 4: Building a Complete App][12]
    
-   [React vs Web Components Comparison][13]
    

## **Web Components Part 1: Your First Custom Element**

This first lesson introduces building user interfaces using Web Components, which let you bundle together HTML, CSS, and JavaScript into a single reusable element.

These elements can be treated just like built-in HTML elements such as `h1`, `div`, and `img`. They're particularly useful when you want to break a page into smaller, self-contained, and reusable parts like headers, tables, or forms, while keeping the code for each organized and isolated.

In this playback, you'll learn:

-   How to create a JavaScript class that represents a custom HTML element
    
-   How to access the attributes of the web component
    
-   How to create and use a web component in HTML
    

The lesson focuses on creating a `LegendHeader` component that can be reused throughout a web application. You'll see how custom elements can have attributes just like regular HTML elements (such as `src` in an `img` tag), and how these attributes can be changed from JavaScript code, triggering methods in response.

**View the playback here:** [**Web Components Part 1- LegendHeader**][14]

## **Web Components Part 2: Data Communication**

Building on the previous lesson, this playback demonstrates how to create multiple components that share data. I'll enhance the `LegendHeader` component to display the count of legends being tracked, and add a new `LegendTable` component that displays all the CS legends in a database using an HTML table.

A key concept introduced in this lesson is having a top-level element hold the web app's data and letting it communicate changes to the components that rely on it. This approach makes component management more organized and maintainable.

In this playback, you'll learn:

-   How to create and work with multiple components
    
-   How to set up and use 'observable attributes' in a web component
    
-   How a top-level element can manage data and inform components when data changes
    

**View the playback here:** [**Web Components Part 2- LegendTable**][15]

## **Web Components Part 3: Custom Events**

This lesson expands the application by adding a `NewLegendForm` component that allows users to add new legends to the database. The playback introduces the concept of custom events that 'bubble' up through the DOM, enabling top-level elements to control requests for data.

You'll learn why it's often better for components not to manage app-wide data themselves. Having a top-level element manage the entire web app's data makes components simpler and more reusable, as they don't need to know about each other or communicate directly.

In this playback, you'll learn:

-   How components can generate custom events to request data instead of managing it themselves
    
-   How a top-level element can listen for events and handle them
    
-   How a top-level element accesses data and passes it to components that need it
    

**View the playback here:** [**Web Components Part 3- NewLegendForm**][16]

## **Web Components Part 4: Building a Complete App**

In this final lesson, I bring everything together to create a complete application with authentication. I'll add a new `AuthBox` component and implement a light authentication system so that only registered, logged-in users can add new legends to the database.

The playback uses sessions on the server to control user access and reinforces the importance of centralized data management in component-based architecture.

In this playback, you'll learn:

-   How to implement user authentication with a web component
    
-   How to integrate all components into a complete, functional application
    
-   Best practices for data management in component-based web applications
    

**View the playback here:** [**Web Components Part 4- AuthBox**][17]

## **React vs Web Components Comparison**

Some of the main differences between React and web components can be summarized like this:

| Key Property | React | Web Components |
| --- | --- | --- |
| **Component Definition** | Uses functions (typically) to define components | Uses JavaScript classes that extend HTMLElement |
| **Tooling Requirements** | Requires tools for installation, transpilation, bundling (npm, webpack, babel, and so on) | Native browser support with no build tools or installation required |
| **Template Syntax** | Uses JSX, an HTML-like syntax within JavaScript | Uses standard HTML in strings or HTML templates |
| **DOM Updates** | Uses Virtual DOM to efficiently batch and minimize actual DOM manipulations | Directly manipulates the DOM, typically less optimized for frequent updates |
| **Property Types** | Accepts various data types as props (strings, arrays, objects, functions) | Only accepts strings as attributes in HTML |
| **Rendering Model** | Declarative: describe what the UI should look like, React handles updates | More imperative: directly manipulate the DOM in response to changes |
| **Style Encapsulation** | No built-in style encapsulation (requires CSS-in-JS or CSS Modules) | Built-in style encapsulation with Shadow DOM |
| **Browser Support** | Works in all browsers via polyfills | Modern browsers only (may require polyfills for older browsers) |
| **Ecosystem** | Large ecosystem with many libraries and tools | Smaller ecosystem, but growing |

## Wrapping Up

These four lessons cover the fundamentals of Web Components, but there's much more to explore. Web Components provide a standards-based way to create reusable elements without the need for external libraries or frameworks, making them particularly valuable for building maintainable and portable code.

If you found this format helpful, explore the rest of the book to see how full web apps are built from scratch using modern tools and approaches.

Web Components represent just one approach to component-based web development. Keep building, keep reading, and try out the other playbacks when you're ready to go further.

If you have feedback about the playbacks I'd love to hear from you. You can reach me here [mark@playbackpress.com][18].

---

![Mark Mahoney](https://cdn.hashnode.com/res/hashnode/image/upload/v1742697935887/aa3553a0-8522-481b-bda8-5bf4e1df7249.png)

[Mark Mahoney][19]

Read [more posts][20].

---

If this article was helpful, share it.

Learn to code for free. freeCodeCamp's open source curriculum has helped more than 40,000 people get jobs as developers. [Get started][21]

ADVERTISEMENT

[1]: /news/tag/web-components/
[2]: /news/author/markm208/
[3]: https://www.freecodecamp.org/news/a-brief-introduction-to-react/
[4]: https://developer.mozilla.org/en-US/docs/Web/API/Web_components
[5]: https://playbackpress.com/books/webdevbook/
[6]: https://playbackpress.com/books/
[7]: https://markm208.github.io/
[8]: https://developer.mozilla.org/en-US/docs/Web/API/Web_components
[9]: #heading-web-components-part-1-your-first-custom-element
[10]: #heading-web-components-part-2-data-communication
[11]: #heading-web-components-part-3-custom-events
[12]: #heading-web-components-part-4-building-a-complete-app
[13]: #heading-react-vs-web-components-comparison
[14]: https://playbackpress.com/books/webdevbook/chapter/3/8
[15]: https://playbackpress.com/books/webdevbook/chapter/3/9
[16]: https://playbackpress.com/books/webdevbook/chapter/3/10
[17]: https://playbackpress.com/books/webdevbook/chapter/3/11
[18]: mailto:mark@playbackpress.com
[19]: /news/author/markm208/
[20]: /news/author/markm208/
[21]: https://www.freecodecamp.org/learn/