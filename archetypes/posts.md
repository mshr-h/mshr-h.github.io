---
title: "{{ replace .Name "-" " " | title }}"
slug: "{{ substr .Name 11 }}"
subtitle:    ""
description: ""
date:        "{{ substr .Name 0 10 }}"
author:      "Masahiro Hiramori"
image:       ""
tags:        []
categories:  ["Tech"]
draft:       false
---
