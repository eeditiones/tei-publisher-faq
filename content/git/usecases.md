---
title: "How to use it?"
date: 2020-12-11T20:35:42+01:00
tags: ["git", "tuttle", "github","gitlab","use case"]
weight: 3
---

There are 2 different scenarios to choose from when configuring your TEI-Publisher project: ___Git to DB___ and ___DB to Git___. 


You cannot
use these modes at the same time but have to decide for one option per TEI-Publisher instance.

{{% notice note %}}
in the pictures below the Github Logo is used but Gitlab can be used the same way
{{% /notice %}}


![Git 2 DB](/images/git2db.png)

### Git as the 'source of truth'

With 'Git to DB' editors will work with their own tools and push changes to Git. 
The data are kept in Git and are deployed to TEI-Publisher
on demand or automatically by an Git action. Data can be updated completely or incrementally just applying
the changes since last update.

There is a simple dashboard to trigger the updating of data and checking the status of connected repositories.

![DB 2 Git](/images/dashboard.png)

When opening the dashboard the status of the configured repositories is requested. The colors of
of the rows signal the status of the respective repository.

| color | Status |
| ----- | ------ |
| orange | new - the repository is configured but has not yet been deployed into TEI-Publisher. The 'incremental' button is disabled as the data need to be checked out before. |
| green | up-to-date - the are no differences between data in the database and the repository. 'Incremental' can be used to trigger update of differences. |
| red | behind - the data in the database are behind the status in the repository. An update is needed |



![DB 2 Git](/images/db2git.png)

### The database as the 'source of truth':
When working with 'DB to Git' authors have the option to push their changes to the repository
directly from document view. In the upper right toolbar there will be a button to push your
changes to the repository.
This is especially useful when working with annotations.
 
 
![toolbar](/images/toolbar.png)
 
Clicking the highlighted icon will pop up a dialog to specify a message for the changes.

![toolbar](/images/commit.png)

Confirming the dialog will push the changes to the repository.


