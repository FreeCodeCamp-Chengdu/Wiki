---
title: Developer Tools secrets that shouldn’t be secrets
authorURL: ""
originalURL: https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/
translator: ""
reviewer: ""
---

Monday, November 1st, 2021 at 3:21 pm

> **Update**: As this [is blowing up on Hackernews](https://news.ycombinator.com/item?id=29071700) I added information to each of the tips in which environment they are supported in parenthesis after each heading. When I state “Chromium browsers”, this refers to all browsers that use the Chromium core and also feature all the Developer Tools. This is Chrome, Microsoft Edge, Brave and many more. As a reminder: Microsoft Edge is the browser that comes with Windows 10/11 and is based on Chromium and thus from a platform perspective simular to Chrome. They differ in UX and services around the core. Edge Developer Tools work closely with Google on bringing the work we add to the product back into the Chromium Core. But some of the things I am talking about here are experiments and exclusively in Microsoft Edge, which is available on Windows, Mac and Linux. Some functionality is only available inside Visual Studio Code via the [Edge DevTools for VS Code extension](https://aka.ms/devtools-for-code) .

This is a talk that I’ve given at [CityJS](https://cityjsconf.org/) this September. I am a principal product manager for developer tools in Microsoft Edge and these are things I encountered during working on the tools, documenting them and going through user feedback.

You can watch the [recording of the talk on Youtube](https://www.youtube.com/watch?v=q_qzHzIVxw4) .

<!-- more -->

Here’s a write-up of all the things I covered:

## 1\. Console is much more than \`log()\`!

(All browsers with developer tools following the standard)

There is no doubt that, besides the Elements tool, Console is the most used part of the browser developer tools. Specificially, people love to debug by putting a \`console.log()\` in their code to learn what’s going on. There are a few problems with that, and there are better ways to debug scripts, but as this is what people do, let’s talk how to make that experience better.

The first problem is log messages that aren’t removed when a product goes live clogging up the Console. Finding the information you’re looking for becomes daunting and the best way to work with that is to learn about the [console filtering options available to you](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-filters) . Using these you can filter the reporting of the console to the things you care about and block out a lot of the noise.

[![filtering options in the console tool](https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_-1024x510.png)](https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_.png)

### What is that you’re logging?

The next problem with using \`console.log()\` is that we seem to only log values and forget to add where they come from. For example, when you use the following code, you get a list of numbers, but you don’t know what is what.

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span>width<span style="color: #009900;">)</span>
console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span>height<span style="color: #009900;">)</span></pre></td></tr></tbody></table>

console.log(width) console.log(height)

The easiest way to work around that issue is to wrap the things you want to log in curly braces. The console then logs both the name and the value of what you want to know about.

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span>width<span style="color: #009900;">}</span><span style="color: #009900;">)</span>
console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span>height<span style="color: #009900;">}</span><span style="color: #009900;">)</span></pre></td></tr></tbody></table>

console.log({width}) console.log({height})

[![Using curly braces around variables in log messages logs their name and their value](https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png)

### Adding to your console vocabulary

[![Examples of warn, info and error messages and how they are displayed in the console](https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png)

In addition to \`console.log()\` you have a [lot more methods you can use](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-log) . For example, \`console.warn()\` logs a warning, \`console.info()\` an informational message, and \`console.error()\` an error message. This not only results in slighty different displays in the console, but it also gives your messages a different log level, which means it is easier to filter for them.

### Errors and assertions in Console

[![The error method of console shows an error, and assert is a shortcut for an if statement with a console.log inside](https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png)

Displaying an error in the console is different to throwing an error, but it still is a good idea to show the severity of an issue to the person maintaining or debugging the product. Another interesting method is \`console.assert()\`, which only logs a message when a certain condition is met. Often you find yourself writing an \`if\` statement with a \`console.log()\` inside. Using \`assert()\` makes that one redundant and you have one less thing to worry about when cleaning up your debugging code.

### Tracing where something came from

[![Example of using console.trace() to track back where a call came from](https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png)

Often you find yourself adding a \`console.log(‘called’)\` or similar to test if a certain functionality is even triggered. Once you have that the next thing you normally want to find out what called that method. That’s what \`console.trace()\` is for, as it doesn’t only tell you that something was called, but also where the call came from.

### Grouping console messages

If you have a lot to log, you can use \`console.group(‘name’)\` and \`console.groupEnd(‘name’)\` to wrap the messages in collapsible and expandable messages in the Console. You can even define if the groups should be expanded or collapsed by default.

[![An example of defining groups in the console](https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png)

### Displaying and filtering lots of information in the console as tables

If you want to display a lot of of information as a log, it can become daunting to read the information. The \`console.table()\` method displays array-like data as a table in the console, and you can filter what you want to display by giving it an array of the properties you want to see.

For example, you can use \`let elms = document.querySelectorAll(‘:is(h1,p,script’)\` to get all H1, paragraph and script elements from the document and \`console.table(elms)\` to display this information as a table. As the different elements have a boatload of attributes and properties, the resulting table is pretty unreadable. If you filter down to what you are interested in by using \`console.table(elms,\[‘nodeName’, ‘innerText’, ‘offsetHeight’\])\` you get a table with only these properties and their values.

[![Code example using console.table() and its filtering options](https://christianheilmann.com/wp-content/uploads/2021/11/Slide14.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide14.png)

The table structure is maintained when you copy and paste this information, which makes it a wonderful tool to get data into Excel or Word, for example.

### Blinging it up: $() and $$()

The console comes with a lot of convenience methods you can use called the [Console Utilities](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/utilities) . Two very useful ones are \`$()\` and \`$$()\` which are replacements for \`document.querySelector()\` and \`document.querySelectorAll()\` respectively. These not only return the nodeList you expect, but also cast the results to arrays, which means you can use \`map()\` and \`filter()\` on the results directly. The following code would grab all the links of the current document and return an Array with objects that contain only the \`href\` and \`innerText\` properties of each link as \`url\` and \`text\` properties.

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">$$<span style="color: #009900;">(</span><span style="color: #3366CC;">'a'</span><span style="color: #009900;">)</span>.<span style="color: #660066;">map</span><span style="color: #009900;">(</span>a <span style="color: #339933;">=&gt;</span> <span style="color: #009900;">{</span>
  <span style="color: #000066; font-weight: bold;">return</span> <span style="color: #009900;">{</span>url<span style="color: #339933;">:</span> a.<span style="color: #660066;">href</span><span style="color: #339933;">,</span> text<span style="color: #339933;">:</span> a.<span style="color: #660066;">innerText</span><span style="color: #009900;">}</span>
<span style="color: #009900;">}</span><span style="color: #009900;">)</span></pre></td></tr></tbody></table>

$$
('a').map(a => { return {url: a.href, text: a.innerText} })

[![An example how the $$ function returns a collection of HTML elements that you can filter like any other array](https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png)

## 2\. You can log without source access – live expressions and logpoints
(Chromium browsers)

The normal way to add a \`console.log()\` is to put it inside your code at the place you want to get the information. But you can also get insights into code you can’t access and change. [Live expressions](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/live-expressions) are a great way to log information without changing your code. They are also incredible to log values that change constantly without flooding the console and thus slowing down your product. You can see the difference in the following screencast:

Logpoints are a special kind of breakpoint. You can right-click any line in a JavaScript in the Sources tool of the Developer Tools and set a logpoint. You get asked to provide an expression you’d like to log and will get its value in the console when the line of code is executed. This means you can technically inject a \`console.log()\` anywhere on the web. I [wrote about logpoints](https://christianheilmann.com/2021/08/24/using-console-log-on-any-website-logpoints-let-you-do-that/) back in August and you can see a demo in the following screencast:

## 3\. You can log outside the browser – VS Code debugger
(Chromium Browsers and VS Code)

When you start a debugging session in Visual Studio Code, you can spawn a browser instance and the Debug Console becomes the Console you are used to from the browser developer tools. I blogged about this in July in detail, so you can [read up there how to do that](https://christianheilmann.com/2021/07/30/using-console-log-debugging-in-visual-studio-code/) . There is also more in the [official documentation](https://docs.microsoft.com/microsoft-edge/visual-studio-code/microsoft-edge-devtools-extension#browser-debugging-with-microsoft-edge-devtools-integration-in-visual-studio-code).

[![](https://christianheilmann.com/wp-content/uploads/2021/10/Slide24-842x1024.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide24.png)

You can also watch this one minute video of me showing the functionality:

## 4\. You can inject code into any site – snippets and overrides.
(Chromium Browsers)

[Snippets](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/snippets) are a way in Developer Tools to run a script against the current web site. You can use the [Console Utilities](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/utilities) in these scripts and it is a great way to write and store complex DOM manipulation scripts you normally execute in the Console. You can run your scripts in the window context of the current document either from the snippets editor or from the command menu. In the latter case, start your command with an ! and type the name of the snippet you want to run.

[Overrides](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/overrides) allow you to store local copies of remote scripts and override them when the page loads. This is great if you have, for example, a slow build process for your whole application and you want to try something out. It is also a great tool to replace annoying scripts from third party web sites without having to use a browser extension.

## 5\. You can inspect and debug much more than you know!
(Chromium Browsers)

You may know the Chromium developer tools from browsers like Google Chrome, Brave or Microsoft Edge, but they are available in a lot more environments. Any app that’s based on Electron can have them enabled and you can use the Tools to peek under the hood and see how the product was done. This works, for example, in GitHub Desktop, Visual Studio Code, or you can even debug the Developer Tools of the browser using Developer Tools!

If you inspect the Developer Tools, you will see that they are written in HTML, CSS and TypeScript. It is an exciting environment to use these technologies, as you you know the rendering engine your code will run in – something you never know on the web.

[![Inspecting the Chromium Developer tools with another instance of the developer tools](https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png)

### Edge Developer Tools in Visual Studio Code
(Microsoft Edge via a VS Code extension)

The embeddable nature of the tools also allowed us to offer you a way to use them outside the browser. The [Microsoft Edge Tools for Visual Studio Code](https://aka.ms/devtools-for-code) extension brings the tools to Visual Studio Code. That way you can use the visual debugging tools right next to your code editor and you don’t need to jump between the two all the time.This also ties in with the “Console in Visual Studio Code” trick mentioned earlier. When you start a debugging session and you click the Developer Tools icon, the tools will open or – the first time – you will be prompted to install the extension.

[![Inspect button in the debug bar of Visual Studio Code](https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png)

[![Microsoft Edge Developer tools open in an instance of Visual Studio Code](https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png)

## 6\. Some dirty secrets…

Working intimately with developer tools and getting feedback and usage information taught me a few dirty secrets. The first one is that whilst we are all super excited about all the amazing features of developer tools, users only use a very small percentage of them. Many things heralded as the best thing since sliced bread in presentations and video tutorials are hardly every opened, let alone used. I thought this was about a lack of documentation and we spent a massive amount of time to update the [DevTools documentation](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/) to ensure everything in them is described and explained, but that wasn’t it. Documentation is something people seem to go to as a last resort when they are stuck and Google/Stack Overflow/Social channels didn’t yield any results.

### Developer tools have become complex and are overwhelming – a few ideas how to fix that
(Microsoft Edge)

It might be that the plain fact is that the Developer Tools of browsers grew organically over the years and can be incredibly overwhelming to look at. And that bothers me and I think we should do better. Here’s my mantra when it comes to tools for developers:

> Developer tools should not expect people to be experts but turn them into experts over time.

We’re working on a few ideas to make that easier, and you will soon see those in Microsoft Edge. One idea we had is a “Focus Mode”. Instead of showing you all the tools and tabs we sorted the tools into different use cases, like “Elements/CSS debugging”, “Sources/JavaScript Debugging” or “Network inspection”. We then show only the relevant tools and hide all the ones that may be confusing or in the way.

[![Developer tools in focus mode, showing only what's needed in the current context](https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png)

Another feature we are working on are “informational overlays”. You get a help button that allows you to turn on overlays for the developer tools, explaining what each of the tools is, how to use it and providing links to the documentation. We hope that this would make it easier for people to learn about more features.

[![Developer tools covered by overlays explaining what each of them are,](https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png)

### There is still a disconnect between authoring code and debugging the outcome
(Microsoft Edge)

Whilst it is amazing what tools provide us these days there is still a disconnect between authoring and debugging. Most of the time we write our code, create the app and then go to the browser to see what doesn’t work. We then use the browser developer tools to tweak and fix these issues. And then comes the big issue we still need to fix: how do you get the changes you created using the browser developer tools back into your code? Most of the time, the answer is “copy and paste or try to remember what needs changing”.

We’re currently working on two ways to make this easier. One is to replace the in-devtools editor with Visual Studio Code when it is available and to change files on the hard drive as you use the browser developer tools. The other is part of the VS Code extension and changes the source code in the editor as you use the developer tools but still gives you the final say in changing the file on disk. I [described the problem and the possible solutions on the Edge blog](https://blogs.windows.com/msedgedev/2021/10/21/improved-authoring-debugging-devtools-visual-studio-code/) or you can watch the following two screencasts to see them in action.

CSS Mirroring in Visual Studio Code:

What if… Visual Studio Code became the editor of in-browser Developer Tools?

## 7\. You’re the audience and the clients of Developer Tools!
(Applies to all browsers, but channels shown here are Microsoft Edge only)

As a developer, you are the main audience for Developer Tools. We are open to your feedback and many of the recent changes to the tools are direct results from demands from outside developers. We try to make this as easy as possible by providing in-context ways to contact us directly. For example, the Visual Studio Code extension has prominent links and buttons for you to report issues and request features.

[![Screenshot of the in-context links provided in the VS Code extension to demand new features, file bugs and learn abour experiments](https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png)

The [source code of the extension](https://github.com/microsoft/vscode-edge-devtools) is also on GitHub and you can [file issues](https://github.com/microsoft/vscode-edge-devtools/issues) there.

The in-browser developer tools also have a direct button to give us feedback. To make it easier for you to provide actionable feedback, the button includes a lot of automatic information.

[![The feedback tool built into the browser developer tools of Microsoft Edge](https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png)

It records automatically what URL the issue happened on, takes a screenshot to include and offers to send diagnostic data. We also ask for you to provide an email in case we need more information and you can add attachments and info how to recreate the issue. We check this feedback daily, and a lot of great inventions and bug fixes came from that source.

[Share on Mastodon (needs instance)](https://mastodon.social/share?text=Developer Tools secrets that shouldn’t be secrets https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/)

[Share on Twitter](http://twitter.com/share?url=https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/&text=Developer Tools secrets that shouldn’t be secrets&via=codepo8)

## My other work:

-   [The Developer Advocacy Handbook](https://developer-advocacy.com)
    -   [Buy it on Amazon](https://www.amazon.com/dp/B0BKNTPDFJ/)
    -   [Buy it on Leanpub](https://leanpub.com/developer-advocacy-handbook)
-   Skillshare Classes:
    -   [Tools and Tips to Optimize Your Workflow as a Developer](https://skl.sh/3uKu5G1)
    -   [Tools for Improving Product Accessibility](https://skl.sh/3eCFWRR)
    -   [The JavaScript Toolkit: Write Cleaner, Faster & Better Code](https://skl.sh/2CpiTGZ)
    -   [Demystifying Artificial Intelligence: Understanding Machine Learning](https://skl.sh/2MHkYl1)

[Skip to search](#s)

[Christian Heilmann](http://christianheilmann.com) is the blog of Christian Heilmann [chris@christianheilmann.com](mailto:chris@christianheilmann.com?subject=NOT%20A%20GUEST%20POST%20REQUEST&body=I%20understand%20this%20blog%20does%20not%20do%20guest%20posts) (Please do not contact me about guest posts, I don't do those!) a Principal Program Manager living and working in Berlin, Germany.

Theme by Chris Heilmann. SVG Icons by [Dan Klammer](https://github.com/danklammer/bytesize-icons) . Hosted by MediaTemple. Powered by Coffee and Spotify Radio.

[Get the feed, all the cool kids use RSS!](https://christianheilmann.com/feed/)
$$
