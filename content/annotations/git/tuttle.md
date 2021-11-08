---
title: "What is Tuttle?"
menuTitle: "What is Tuttle?"
date: 2020-12-11T20:35:42+01:00
tags: ["git", "tuttle"]
weight: 1
---

![Tuttle logo](/images/HPTuttle-1866.png)

***Tuttle*** adds version control to data collections in eXist-db applications such as TEI Publisher.

It is part of and published by the [e-editiones society](https://e-editiones.org/) and can be used alongside TEI Publisher to allow synchronization
of data collections between TEI Publisher instances and a Git repository. 

***Tuttle*** wraps the APIs of the Git service providers under a common OpenAPI in TEI Publisher allowing
projects to seemlessly add version control to their data.


## Modes of operation

Tuttle offers 2 modes of operation:
* 'Git2DB'
* 'App2Git'

{{% notice note %}}
in the pictures below the Github Logo is used but Gitlab can be used the same way
{{% /notice %}}

### Git to DB

In this scenario users use their own tools to edit documents and push their changes to a git repository.

With Tuttle enabled the configured eXist-db collection can be updated completely or incrementally on request
or by using a git workflow for automatic deployment.

![DB 2 Git](/images/git2db.png)

### App to Git

An application may choose to use the Tuttle application to synchronize their database collections to a Git repository and allow
users without Git knowledge to version their documents. 

![DB 2 Git](/images/app2git.png)

In this case the application will use the Tuttle API to offer a custom integration into its user interface.


