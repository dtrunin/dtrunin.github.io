---
layout: post
title: "Critical Rendering Path"
---

The Critical Rendering Path is the sequence of steps the browser goes through to convert the HTML, 
CSS, and JavaScript into pixels on the screen.

### Document Object Model (DOM)

The browser processes HTML markup, and builds the DOM tree.
The DOM contains all the content of the page.

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

### Executing JavaScript

When the HTML parser encounters a script tag, it pauses its process of constructing the DOM 
and yields control to the JavaScript engine; after the JavaScript engine finishes running, 
the browser then picks up where it left off and resumes DOM construction.

When the HTML parser encounters a script tag, it behaves depending on the `async` and `defer` script attributes.
If the async attribute is present, then the script will be executed asynchronously, as soon as it is loaded. 
If the async attribute is not present, but the defer attribute is present, 
then the script is executed when the page has finished parsing. 
If neither attribute is present, then the script is fetched and executed immediately, 
before the browser continues parsing the page.

JavaScript is executed at the exact point where it is inserted in the document.
JavaScript blocks DOM construction unless explicitly declared as async.
