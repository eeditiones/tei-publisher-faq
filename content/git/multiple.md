---
title: "Can i have multiple repositories?"
menuTitle: "Multiple repositories"
date: 2021-11-03T13:32:17+01:00
tags: ["git","repository", "Github","Gitlab"]
weight: 5
---

Yes. It's possible to configure multiple repositories even of different types.

So for instance you can have one Gitlab repository connected to one target collection in the database and another
collection that is bound to a Github repository somewhere else.

Each configuration item will correspond to exactly one target collection. However target collections
must not be nested as that would cause trouble. 
