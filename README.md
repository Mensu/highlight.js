# Highlight.js


Highlight.js is a syntax highlighter written in JavaScript. It works in
the browser as well as on the server. It works with pretty much any
markup, doesn’t depend on any framework and has automatic language
detection.

Note: This branch is edited by [``Mensu``](https://github.com/Mensu) to support line numbers.

## Compile from Source

To compile, you should have ``node.js`` installed first. Then run ``npm install`` to install dependent packages

Generally speaking, running ``node ./tools/build.js`` provides compiled files

There are several useful options:

- ``-n`` get uncompressed version
- ``-t`` specify target platform. Four expected choices: ``browser``, ``cdn``, ``node``, ``all``
- ``:common`` to support commonly used languages
- ``lang1 lang2 ...`` command ``node ./tools/build.js python less`` would install python and less support



``npm start`` is equivalent to ``node ./tools/build.js -n :common``, namely supporting commonly used languages without compressing the generated files

Before compilation you are supposed to choose a theme you are fond of and [edit its css files](#edit-css-files) in ``./src/style`` so as to make line numbers visible. I have done this for ``default`` and ``monokai-sublime`` styles, which you might want to refer to when you have trouble with editing css files

After compilation you may checkout ``./build`` folder and get ``highlight.pack.js``

The generated ``highlight.pack.js`` can be previewed by opening ``./build/demo/index.html`` in a browser

### Edit CSS Files

- First Step: find selector ``.hljs``, replace the following rules

```css
.hljs {
  display: block;
  overflow-x: auto;
  ...
}
```

with

```css
.hljs {
  display: inline-block;
  min-width: 100%;
  box-sizing: border-box;
  border-radius: 3px;
  ...
}
```

namely:

```css
.hljs {
  /* display: block; */
  /* overflow-x: auto; */
  display: inline-block;
  min-width: 100%;
  box-sizing: border-box;
  border-radius: 3px;
  ....

}
```

- Last Step: paste the following rules into the CSS file

```css
.hljs {
  counter-reset: lines;
}
.hljs .line {
  counter-increment: lines;
}
.hljs .line::before {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;

  -o-user-select: none;
  user-select: none;

  content: counter(lines); text-align: right;
  display: inline-block; width: 2em;
  padding-right: 0.5em; margin-right: 0.5em;
  color: #BBB; border-right: solid 1px;
}
.hljs-comment ~ .line {
  color: #75715e;
}
.hljs-comment ~ .line::before {
  font-style: normal;
}
```

## Getting Started

### Recommended

```javascript
// highlightInit.js
hljs.configure({noLinenum: false});  // default to false. unnecessary here
// get the DOM elements to highlight
var selector = 'fill in with the selector of the code blocks to highlight';
var codeBlocks = document.querySelectorAll(selector);
// call hljs.highlightBlock to highlight
Array.prototype.slice.call(codeBlocks).forEach(function(oneBlock) {
  hljs.highlightBlock(oneBlock);
  // or
  // hljs.addLinenum(oneBlock); // which simply adds line numbers without highlighting
});
```

```html
<head>
  ...
  <link rel="stylesheet" href="/path/to/styles/edited-style.css">
  ...
</head>
<body>
  ...
  <!-- elements to highlight -->
  ...
  <script src="/path/to/highlight.pack.js"></script>
  <script src="/path/to/highlightInit.js"></script>
  ...
</body>
```

### Original Doc

The bare minimum for using highlight.js on a web page is linking to the
library along with one of the styles and calling
[`initHighlightingOnLoad`][1]:

```html
<link rel="stylesheet" href="/path/to/styles/default.css">
<script src="/path/to/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

This will find and highlight code inside of `<pre><code>` tags; it tries
to detect the language automatically. If automatic detection doesn’t
work for you, you can specify the language in the `class` attribute:

```html
<pre><code class="html">...</code></pre>
```

The list of supported language classes is available in the [class
reference][2].  Classes can also be prefixed with either `language-` or
`lang-`.

To disable highlighting altogether use the `nohighlight` class:

```html
<pre><code class="nohighlight">...</code></pre>
```

## Custom Initialization

When you need a bit more control over the initialization of
highlight.js, you can use the [`highlightBlock`][3] and [`configure`][4]
functions. This allows you to control *what* to highlight and *when*.

Here’s an equivalent way to calling [`initHighlightingOnLoad`][1] using
jQuery:

```javascript
// hljs.configure({noLinenum: true});  // this setting is hide the line numbers
$(document).ready(function() {
  $('pre code').each(function(i, block) {
    // hljs.addLinenum(block); // simply add line numbers without highlighting
    hljs.highlightBlock(block);
  });
});
```

You can use any tags instead of `<pre><code>` to mark up your code. If
you don't use a container that preserve line breaks you will need to
configure highlight.js to use the `<br>` tag:

```javascript
hljs.configure({useBR: true});

$('div.code').each(function(i, block) {
  hljs.highlightBlock(block);
});
```

For other options refer to the documentation for [`configure`][4].


## Web Workers

You can run highlighting inside a web worker to avoid freezing the browser
window while dealing with very big chunks of code.

In your main script:

```javascript
addEventListener('load', function() {
  var code = document.querySelector('#code');
  var worker = new Worker('worker.js');
  worker.onmessage = function(event) { code.innerHTML = event.data; }
  worker.postMessage(code.textContent);
})
```

In worker.js:

```javascript
onmessage = function(event) {
  importScripts('<path>/highlight.pack.js');
  var result = self.hljs.highlightAuto(event.data);
  postMessage(result.value);
}
```


## Getting the Library

You can get highlight.js as a hosted, or custom-build, browser script or
as a server module. Right out of the box the browser script supports
both AMD and CommonJS, so if you wish you can use RequireJS or
Browserify without having to build from source. The server module also
works perfectly fine with Browserify, but there is the option to use a
build specific to browsers rather than something meant for a server.
Head over to the [download page][5] for all the options.

**Don't link to GitHub directly.** The library is not supposed to work straight
from the source, it requires building. If none of the pre-packaged options
work for you refer to the [building documentation][6].

**The CDN-hosted package doesn't have all the languages.** Otherwise it'd be
too big. If you don't see the language you need in the ["Common" section][5],
it can be added manually:

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.4.0/languages/go.min.js"></script>
```

**On Almond.** You need to use the optimizer to give the module a name. For
example:

```
r.js -o name=hljs paths.hljs=/path/to/highlight out=highlight.js
```


## License

Highlight.js is released under the BSD License. See [LICENSE][7] file
for details.

## Links

The official site for the library is at <https://highlightjs.org/>.

Further in-depth documentation for the API and other topics is at
<http://highlightjs.readthedocs.io/>.

Authors and contributors are listed in the [AUTHORS.en.txt][8] file.

[1]: http://highlightjs.readthedocs.io/en/latest/api.html#inithighlightingonload
[2]: http://highlightjs.readthedocs.io/en/latest/css-classes-reference.html
[3]: http://highlightjs.readthedocs.io/en/latest/api.html#highlightblock-block
[4]: http://highlightjs.readthedocs.io/en/latest/api.html#configure-options
[5]: https://highlightjs.org/download/
[6]: http://highlightjs.readthedocs.io/en/latest/building-testing.html
[7]: https://github.com/isagalaev/highlight.js/blob/master/LICENSE
[8]: https://github.com/isagalaev/highlight.js/blob/master/AUTHORS.en.txt
