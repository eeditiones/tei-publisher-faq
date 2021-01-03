---
title: "How can I upgrade the version of webcomponents I'm using?"
menuTitle: "Upgrading webcomponents"
date: 2021-01-03T12:14:39+01:00
tags: ["webcomponents", "versions", "install"]
---

TEI Publisher uses a separate library for the webcomponents, which are the basic building blocks of the user interface. When you install a certain version of TEI Publisher, it will by default use the version of the webcomponent library which was current at the time it was released. You can check which version you are using by looking at the footer of the TEI Publisher entry page (or an app you generated from TEI Publisher):

![Version screenshot](/images/publisher-version.png)

From the screenshot above we know that this instance is using version *1.14.0* of the webcomponents library and runs version *1.0.0* of the server-side API. You can check if a newer version is available for the webcomponents library on the [npm registry](https://www.npmjs.com/package/@teipublisher/pb-components). At the time of writing, there's a newer version, *1.14.1*, available, so we can upgrade our local TEI Publisher instance as follows. The process is the same for generated apps as they use exactly the same configuration mechanism:

1. open the central configuration file for TEI Publisher: `modules/config.xqm` in eXide (or a text editor of your choice).
2. find the variable `$config:webcomponents` and change it to point to the new version
    ```xquery
    declare variable $config:webcomponents :="1.14.1";
    ```
3. save the file. If you edited it in eXide, your changes will come into effect immediately after reloading the page. If you changed it in a local file, you need to build and redeploy your `.xar` package.

{{% notice note %}}
We use *semantic versioning* for the libraries, which means that versions starting with the same major version number will be backwards compatible. You can thus use any version in the *1.x.x* range with any version of TEI Publisher 6 and 7.
{{% /notice %}}