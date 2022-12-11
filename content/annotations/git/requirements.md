---
title: "What is needed to use Tuttle?"
menuTitle: "Requirements"
date: 2021-11-03T15:15:41+01:00
tags: ["requirements","setup","Github","Gitlab"]
weight: 2
---


### 1. Access to a repository

To use Tuttle access to a repository on Github or Gitlab is necessary. This can be public or private repositories.
On-premise servers will of course also work.

### 2. Access token

In order to authenticate to the VCS (version control system) and being allowed to access its API a 'personal access token'
is needed. This can be obtained from the respective service-provider.
 
See ['how to get an auth token'](/git/auth/)

### 3. Tuttle application

Tuttle is a separate eXist-db app and can be installed from the public repository of eXist-db by using the Dashboard app.

### 4. Data collection

The data collection that is referred to in the Tuttle configuration must exist (even if empty) before you can
deploy data to it. Otherwise Tuttle will fail to deploy the data.

