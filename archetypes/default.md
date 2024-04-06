---
title: "{{ replace .Name "-" " " | title }}"
slug: {{ substr (md5 (printf "%s%s" .Date (replace .TranslationBaseName "-" " " | title))) 4 8 }}
date: {{ .Date }}
tags: []
categories: [""]
featured_image: 
description: 
draft: true
sticky: false
---

