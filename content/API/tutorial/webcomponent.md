---
title: "Combining Endpoint and Webcomponent"
menuTitle: "Webcomponents"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 4
---


We now have the endpoint ready, so the next step is to actually create an HTML page and plug it in. First, check if you have the [correct version](/webcomponents/version-upgrade) of the tei-publisher-components library (> 1.32.0).

Next, create a file below `templates`, e.g. `templates/places.html` and paste in the following HTML:

```html
<?xml version="1.0" encoding="UTF-8"?>
<html>
    <head>
        <meta charset="utf-8"/>
        <meta name="viewport" content="width=device-width, minimum-scale=1, initial-scale=1, user-scalable=yes"/>
        <link rel="shortcut icon" type="image/x-icon" href="resources/images/favicon.ico"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-16.png" sizes="16x16"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-24.png" sizes="24x24"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-32.png" sizes="32x32"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-64.png" sizes="64x64"/>

        <title data-template="config:app-title"></title>
        <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Oswald"/>
        <link rel="stylesheet" href="resources/css/theme.css" />
        <script type="module" src="pb-components-bundle.js" data-template="pages:load-components"></script>
        <style>
            main {
                max-width: 41rem;
                margin: 0 auto;
            }
        </style>
    </head>
    <body>
        <pb-page data-template="pages:pb-page" unresolved="unresolved">
            <app-header-layout>
                <app-header slot="header" reveals="reveals" fixed="fixed" effects="waterfall">
                    <app-toolbar class="menubar">
                        <a href="${app}/index.html" class="logo" data-template="lib:parse-params"></a>
                    </app-toolbar>
                </app-header>
                <main>
                    <pb-custom-form id="options" auto-submit="paper-input,paper-icon-button">
                        <paper-input id="query" name="search" label="Suche">
                            <paper-icon-button icon="search" slot="suffix"></paper-icon-button>
                        </paper-input>
                    </pb-custom-form>
                    <pb-split-list url="api/places" subforms="#options" selected="A"></pb-split-list>
                </main>
            </app-header-layout>
        </pb-page>
    </body>
</html>
```

Navigate to the new page by entering [/tei-publisher/places.html](http://localhost:8080/exist/apps/tei-publisher/places.html):

![places.html](/images/places.png)