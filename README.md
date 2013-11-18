# Nerio

Nerio is a safe subset of JavaScript, not unlike <a href="#">Adsafe</a>, that helps you run untrusted code without allowing it access to various language and browser features.

# Background

Sometimes you'd like to allow third-party code to run as part of a page or application. Unfortunately, it's hard to know what nefarious things someone else's code might do. Nerio defines a subset of JavaScript and provides some simple tools and models for checking that an arbitrary piece of JavaScript code is valid Nerio. Static analysis has the benefit that once code is checked, it can run at full speed whether in the browser or a server-side engine. Alternatives involving JavaScript interpreters compiled to <a href="http://asmjs.org">asm.js</a> can be two orders of magnitude slower. WebWorker sandboxes require a virtual DOM, which is also potentially very slow.

# Getting started

To get started with Nerio you should have <a href="http://nodejs.org">Node.js</a> installed on your system. 

```bash
npm install -g nerio
```

Then you can use <code>nerio</code> on the command line, or the programmatic API:

```javascript
nerio = require('nerio');
nerio.check(['js/script-to-check.js'], callback);
```

There is also an API compiled for use directly in the browser.

```html
<script src="//nerio/dist/nerio.min.js"></script>
<script>
	var code = '...';
	nerio.check(code, callback); 
</script>
``` 