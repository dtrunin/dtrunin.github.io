---
layout: post
title: "Critical Rendering Path"
---

The Critical Rendering Path is the sequence of steps the browser goes through to convert the HTML,
CSS, and JavaScript into pixels on the screen.

### Document Object Model (DOM)

In order to render HTML content the browser convert HTML markup into its object-oriented
representation known as the "Document Object Model" (DOM).

In order to construct the DOM the browser performs tokenization and parsing.
The tokenizer consumes character from an HTTP response with the HTML content and emits the following tokens:
DOCTYPE, start tag, end tag, comment, character, end-of-file.
When a token is emitted, it must immediately be handled by the parser.
The parser receives tokens from the tokenizer and modifies or extends document's DOM tree.

The browser won't render any processed content until the DOM is constructed.

```html
<!DOCTYPE html>
<html>
  <head>
    <link href="style.css" rel="stylesheet">
  </head>
  <body>
    <p>Hi <span>display none</span> there!</p>
    <div><img src="awesome-photo.jpg"></div>
  </body>
</html>
```

![DOM](/assets/images/dom.svg)

### CSS Object Model (CSSOM)

The browser during creating DOM receives all CSS stylesheets of the page,
and converts them into a tree structure known as the "CSS Object Model" (CSSOM).

```css
body { font-size: 16px }
p { font-weight: bold }
span { color: red }
p span { display: none }
img { float: right }
```

![CSSOM](/assets/images/cssom.svg)

CSS is a render blocking resource - the browser won't render any processed content
until the CSSOM is constructed.

#### Conditional CSS

We can speed up page rendering if declare styles that are not applied to the rendering page
as non-blocking.
CSS "media types" and "media queries" allow us to apply CSS styles only under certain conditions.
The browser still downloads the non-blocking CSS assets, but with a lower priority.

```html
<!-- No media type or query, so it applies in all cases and always render blocking -->
<link href="style.css" rel="stylesheet">
<!-- Applies only when the content is being printed, not render blocking -->
<link href="print.css" rel="stylesheet" media="print">
<!-- The browser executes "media query", if the conditions match - render blocking  -->
<link href="other.css" rel="stylesheet" media="(min-width: 40em)">
```

### Render Tree

The browser combines the DOM and CSSOM into a "render tree",
which captures all the visible DOM content on the page
and all the CSSOM style information for each node.

![Render Tree](/assets/images/render-tree.svg)

### Layout

The layout step determines where and how the elements are positioned on the page,
determining the width and height of each element, and where they are in relation to each other.

### Paint

Paint the individual nodes to the screen.

### Scripts

JavaScript can modify the rest of the document content using the `document.write()` method.
As a result if during DOM construction the browser encounters a script tag it is forces
to stop parsing the remainder of the document until the script is fetched and executed.

JavaScript can query and modify the CSSOM, hence CSS is blocking resource for JavaScript,
and before executing JavaScript from a script tag the browser waits until all styles
preceding the script tag are fetched.

#### Attributes

The `async` attribute determines that the script will be fetched in parallel
to parsing and evaluated as soon as it is available.

The `defer` attribute indicates that the script is executed after the document has been parsed,
but before firing DOMContentLoaded.

The `async` and `defer` attributes remove scripts from the critical rendering path.

#### Events

The `Document.DOMContentLoaded` event fires when the initial HTML document with scripts has been completely loaded and parsed,
without waiting for stylesheets, images, and subframes to finish loading.

The `Window.load` event is fired when the whole page has loaded,
including all dependent resources such as stylesheets and images.

The `Document.readystatechange` event is fired when the `Document.readyState` attribute of a document has changed.
The `Document.readyState` property describes the loading state of the document.
The readyState of a document can be one of following:

- `loading` - the document is still loading.
- `interactive` - the DOM tree is created, the `DOMContentLoaded` event is about to fire.
- `complete` - the document and all sub-resources have finished loading, the `load` event is about to fire.

### Preloading Resources

The tag `<link rel="preload">` is intended for preloading resources that will be used
for the rendering of the current page later.

The tag `<link rel="prefetch">` is intended for prefetching resources that will be used in the next
navigation/page load (e.g. when you go to the next page).
browsers will give prefetch resources a lower priority than preload ones.

The browser stores preloaded/prefetched resources into the cache, and reuses them on future requests.

### Speculative Rendering (Preload Scanning)

The HTML parser stops while fetching a script.
If an HTML page has subsequent external resource, the browser starts loading them only
when the current blocking script is retrieved.
As a result subsequent resources are not loading on parallel.
This causes a delay in the content rendering.
To mitigate this delay the browser performs preload scanning, also called speculative rendering.

The Browser makes a suggestion, that scripts on the page are not going to use the `document.write()` method
to modify the remaining content.
In this case while is blocked on a script, the browser starts a side parser
that scans the rest of the document looking for other resources
e.g. stylesheets, scripts, images etc., and starts loading them.
The browser still executes scripts in the order they appear in the document,
not in order they are fetched.

The upside is that when a speculation succeeds we reduce the resources loading time and the content rendering time.
The downside is that when a speculation fails all its work becomes useless.

![Critical Rendering Path](/assets/images/critical-rendering-path.png)

1. The browser sarts fetching `script2.js` not waiting while `script1.js` is received because of speculative parsing.
2. Scripts are executed in the order they appear in the document. 
`script2.js` is fetched earlier than `script1.js` but it is still executed after `script1.js`.
3. Styles preceding a script should be fetched before the script is executing.
`script3.js` is executed only when `header.css` is received.
4. When scripts and applied to a page received, browser renders the page.
In our example `print.css` isn't applied to the page, and the browser doesn't wait the style is fetched to paint the page.
Also browser doesn't wait fetching async and defer scripts.
5. When all scripts fetched and executed the `Document.DOMContentLoaded` occurs.
6. When the whole page has loaded, including all dependent resources such as stylesheets and images,
the `Window.load` event is triggered. The browser doesn't wait for lazy loading images. 

### References / Further Reading
[https://developer.mozilla.org/en-US/docs/Glossary/speculative_parsing](https://developer.mozilla.org/en-US/docs/Glossary/speculative_parsing)<br>
[https://developers.google.com/web/fundamentals/performance/critical-rendering-path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)<br>
[https://bugzilla.mozilla.org/show_bug.cgi?id=364315](https://bugzilla.mozilla.org/show_bug.cgi?id=364315)<br>
[http://gent.ilcore.com/2011/01/webkit-preloadscanner.html](http://gent.ilcore.com/2011/01/webkit-preloadscanner.html)<br>
