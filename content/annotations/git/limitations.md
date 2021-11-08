---
title: "What are the limitations?"
menuTitle: "Limitations"
date: 2020-12-11T20:35:42+01:00
tags: ["git", "tuttle", "limitations"]
weight: 6
---

### 'DB to Git' does not support concurrent working on a single document.

This mode is targetted towards the common use case of one person
editing a document exclusively. As concurrent access to the same resource may cause
conflicts in version control this should generally be avoided for seemless operation.
 
{{% notice note %}}
Even if conflicts occur the data are always safe and can be retrieved all the time by a
technical person with Git knowledge.
{{% /notice %}}
