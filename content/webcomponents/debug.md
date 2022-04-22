---
title: "Changing and Debugging Webcomponents"
menuTitle: "Debugging / Local Development"
date: 2022-04-22T18:22:38+02:00
tags: ['webcomponents']
---

Sometimes you may want to debug a problem related to one of TEI Publisher's web components - or you may want to improve a component and test it directly within the TEI Publisher application context.

### Prerequisites

In order to debug or test you need to have a local version of TEI Publisher's web component library:

1. on your local system you need [nodejs](https://nodejs.org/en/) and Node's package manager, npm, installed
1. if you have not already done so, clone the [TEI Publisher components](https://github.com/eeditiones/tei-publisher-components) library
2. change into the cloned directory and run `npm install` once
3. start a local server with `npm start`

The last step will bundle the javascript library, generate documentation and then launch a webserver (on port 8000 by default) to serve the sources. It also tries to open the documentation page in your browser if possible. The final output of the `npm start` command should look similar to the following:

```text
es-dev-server started on http://localhost:8000
  Serving files from '/Users/wolfgang/Source/apps/tei-publisher/pb-components'.
  Opening browser on '/'
  Using history API fallback, redirecting route requests to '/api.html'
  Using auto compatibility mode, transforming code on older browsers based on user agent.
```

### Change `config.xqm`

To make the browser load the javascript library files from your local installation, open [modules/config.xqm](https://github.com/eeditiones/tei-publisher-app/blob/49dbcc0bc454c627c564588a9649a3f98bc589c1/modules/config.xqm#L49) in either: the TEI Publisher application you want to debug or a derived application you generated from TEI Publisher.

Search for the two variables: `$config:webcomponents` and `$config:webcomponents-cdn`. Set the first to `dev` instead of specifying a version. The second needs to point to the local server you started with npm above:

```xquery
declare variable $config:webcomponents := "dev";

declare variable $config:webcomponents-cdn := "http://localhost:8000";
```

### Test it

Reload the TEI Publisher page (or a page in your custom application) by holding the ~Shift~ key and pressing the reload button in your browser. The javascript components should now be loaded from your local server instead of the default CDN. 

You can confirm this by opening the ~developer tools~ in your browser, switching to the ~network~ tab and checking the `.js` files being loaded: instead of a few large files, you should now see many small files being loaded, e.g. `src/pb-components-bundle.js` and many more.

Those files are the actual source files, which means you can use the browser's own javascript debugger in the ~developer tools~, set breakpoints and inspect variables. Below you see a screenshot of Chrome's debugger stopping on a breakpoint in `pb-load.js`:

![Chrome debugger](/images/chrome-debug.png)

You can also make changes to any of the source files in your clone of `tei-publisher-components` and - while `npm start` is running - changes will apply whenever you reload the page.