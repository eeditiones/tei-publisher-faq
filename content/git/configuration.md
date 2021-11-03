---
title: "How to configure Tuttle?"
menuTitle: "Configuration"
date: 2020-12-11T20:35:42+01:00
tags: ["git", "tuttle", "github","gitlab","configuration"]
weight: 4
---
Tuttle is configured in `modules/config.xqm` in the TEI-Publisher instance.

Below is an example configuration that shows an example for either Github and Gitlab. Due to the 
different API of these services the configuration differs sligtly. See parameter description
below.




```
(:~
 : Git configuration
 :)
declare variable $config:collections := map {
            "sample-collection-github" : map {
                "vcs" : "github",
                "baseurl" : "https://api.github.com/",
                "ref" : "master",
                "token" : "146152be6a9xxxxxxxxxxx71da8f3e4faf263",
                "hookuser" :  "admin",
                "hookpasswd" : ""
                "owner" : "Jinntec",
                "repo" : "tuttle-demo"
            },
            "sample-collection-gitlab" : map {
                "vcs" : "gitlab",
                "baseurl" : "https://gitlab.com/api/v4/",
                "ref" : "master",
                "token" : "9vq6jmzxxxxxxxxxxobUfoM",
                "hookuser" :  "admin",
                "hookpasswd" : "",
                "project-id" : "25852323"
            }
        };


```

### Common configuration parameters

| Configuration Parameter | Description |
| ----------------------- | ------------ |
| vcs | can be either 'github' or 'gitlab' and signifies the type of repository |
| baseurl | the base URL of the Git Service e.g. 'https://api.github.com | 
| ref | the name of the branch to use |
| token | the authorization token for the Git service. This needs to be obtained for the respective service prior to operation. |
| hookuser | the user name used to talk to the service. | 
| hookpasswd | the password for the hookuser |


### Github 

| Configuration Parameter | Description |
| ----------------------- | ------------ |
| owner | the owning organisation in Github |
| repo | the repository name within the organisation |

The `baseurl` plus the above parameters are used to construct the URL to access the Github repository.

### Gitlab 

| Configuration Parameter | Description |
| ----------------------- | ------------ |
| project-id | Gitlab identifier for the repository. Gitlab is using this id when being accessed via its API. It is listed on the homepage of the repository. | 

`baseurl` plus `project-id` are used to construct the URL to access the Gitlab repository.
