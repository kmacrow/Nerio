<div align="center">
<img src="https://dl.dropboxusercontent.com/u/55111805/nerio.png" style="width:160px" />
</div>

<div align="center">
<b>Nerio</b> is a "safe" subset of JavaScript, not unlike <a href="#">Adsafe</a>, that helps you run untrusted code without<br />allowing it access to various language and browser features.
</div>

# Background

Sometimes you would like to allow third-party code to run as part of a page or application. Unfortunately, it's hard to know what nefarious things someone else's code might do. Nerio defines a subset of JavaScript and provides some simple tools for checking that an arbitrary piece of JavaScript code is valid Nerio code. Static analysis has the benefit that once code is checked, it can run at full speed whether in the browser or a server-side engine. Alternatives involving JavaScript interpreters compiled to <a href="http://asmjs.org">asm.js</a> can be two orders of magnitude slower. WebWorker sandboxes require a virtual DOM, which is also potentially very slow.

# Overview

Nerio provides a simple command line tool as well as several programmatic interfaces for statically checking fragments of JavaScript in memory or files. Nerio checks for use of features or APIs that might enable exfiltration of private data, communication with untrusted servers or attempts to interpose on core browser APIs. Much like a compiler, Nerio can provide detailed line-level warnings and errors when it detects unsafe code. As a pure JavaScript implementation, the entire tool and API can be used from a web page, or in standalone applications via <a href="http://nodejs.org">node.js</a>.

# Implementation

Nerio is implemented entirely in JavaScript. The excellent <a href="http://esprima.org">Esprima</a> parsing engine is used to parse raw JavaScript code into the standard Mozilla <a href="https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey">SpiderMonkey</a> AST data structure. <a href="https://github.com/Constellation/esmangle">esmangle</a> is used to "compress" (remove redundancy, dead code, and normalize syntax in) the AST structure. Nerio then makes several passes over the AST looking for function call sites, identifiers, and symbols that look suspicious. Nerio does not fail early: instead it looks at all of the code and reports any problems found, which is potentially useful for debugging repeated violations or to derive a sense of just how problematic the code is. Internally, Nerio is designed to be modular: modules that implement different checks register themselves with a central engine which orchestrates their execution over the AST and collects violations in a standard way. We are also working on some experimental code-rewriting features in which we rewrite the AST on the fly to wrap suspicious code in runtime checks and then use <a href="https://github.com/Constellation/escodegen">escodegen</a> to generate JavaScript from the transformed AST. 


# Related work

There is a substantial body of work related to Nerio, from the research community as well as industry and open source. Below is a brief summary of several key projects. <i>Gatekeeper</i> in particular enumerates a lot of the JavaScript language and browser API features that we are interested in restricting or disabling altogether. <i>Treehouse</i> provides a lightweight virtual DOM and runtime isolation (via <a href="https://developer.mozilla.org/en-US/docs/Web/Guide/Performance/Using_web_workers">WebWorkers</a>) that we will likely combine with Nerio in future work.     

<b>Virtual runtimes</b>

<i>js.js</i> - <a href="https://github.com/jterrace/js.js/">GitHub page</a><br /> 
A JS interpreter in JS, so you can sandbox while you sandbox. 

<i>Treehouse</i> - <a href="https://www.usenix.org/system/files/conference/atc12/atc12-final159.pdf">ATC '12 paper</a>
Loads a "broker" into a WebWorker. Interposes on browser APIs, freezes them, then runs code against a virtual DOM via message passing with main thread.

<b>Static analysis</b>

<i>FBJS</i> - <a href="https://developers.facebook.com/blog/post/189/">Developer page</a>
Facebook App JS subset (proprietary)

<i>Gatekeeper</i> - <a href="http://research.microsoft.com/pubs/81084/usenixsec09a.pdf">USENIX SEC '09 paper</a>
Static analysis + code rewriting to safe subset of JS

<i>Adsafety</i> - <a href="http://cs.brown.edu/research/plt/dl/adsafety/v1/adsafety.pdf">Website</a>
Verifying static checkers (in this case Adsafe)

<i>Adsafe</i> - <a href="http://www.adsafe.org">adsafe.org</a> (<a href="https://github.com/douglascrockford/ADsafe">GitHub page</a>)<br />
A subset of JS safe for embedding in ads

<i>Browsershield</i> - <a href="http://research.microsoft.com/en-us/projects/shield/bshield-osdi2006.pdf">OSDI '06 paper</a>
Complete JS/HTML rewriting to safe widgets 

<i>Websandbox</i> - <a href="http://www.websandbox.org">websandbox.org</a>
Similar to Browsershield (also by MS), rewrites all web content into safe widgets

<i>Caja</i> - <a href="https://code.google.com/p/google-caja/">Google Code</a>
Ca-pabilities based Ja-vascript. Weird, supposed to pronounce as “Caha” from the Spanish “safe” or “box”. Rewrites html, css, JavaScript to a safe version.


# Conclusions

So far Nerio is a humble prototype, only a shadow of what we have planned for it. That said, the flexibility and effectiveness of our prototype is promising and we hope to continue building it out into a more sophisticated and practicable tool as future work. Static analysis at the AST level is quite difficult, and especially so for a dynamic, mixed-paradigm language like JavaScript. We hope that eventually Nerio will enable a class of browser-based applications that can make stronger guarantees about where our personal data can and cannot end up. 


# Getting started

To get started with Nerio you should have the latest <a href="http://nodejs.org">Node.js</a> (including <code>npm</code>) installed on your system. There is a <code>devenv</code> script included that will install it for you on unix-like platforms. Note: you need to download <code>node</code> and <code>npm</code> directly from <a href="http://nodejs.org">nodejs.org</a> becuase the version tracked by most package managers is too old.  

```bash
npm install -g nerio
```

Then you can use <code>nerio</code> on the command line,

```bash
$ echo "var PI = 3.14;" | nerio
```

or pass it a file to check:

```bash
$ nerio samples/eval.js
Failed:
EVAL: Explicit call to eval() at 2:0
EVAL: Assignment to result of eval() at 4:8
EVAL: Assignment to result of eval() at 4:22
...
```

or the programmatic API:

```javascript
nerio = require('nerio');
nerio.check_code(['js/script-to-check.js'], function(success, err) {
	if( !success )
		console.log(err);
	else
		console.log('Success!');
});
```

There is also an API compiled for use directly in the browser.

```html
<script src="//nerio/dist/nerio.min.js"></script>
<script>
	var code = '...';
	nerio.check_code(code, function(success, err) { ... }); 
</script>
``` 

