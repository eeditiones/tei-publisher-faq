+++
title = "Page view doesn't work"
date =  2020-12-16T22:15:07+01:00
tags = ["navigation", "pb-view"]
+++

Displaying documents page by page requires that page breaks are encoded. In TEI you are expected to explicitly place `pb` elements where each page begins.

Please note that certain vocabularies, like DocBook, do not recognize the concept of a page at all, so page view cannot possibly work for these.