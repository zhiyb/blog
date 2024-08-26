---
title: C# project locale
tags:
  - locale
id: '16'
categories:
  - - C#
  - - c
    - Vixen
date: 2016-12-13 03:00:49
---

If the system locale was set to something different than English, special characters in some source files may cause strange compilation failures.

Explicit `CodePage` settings need to be applied to some projects.

Add

<CodePage>28591</CodePage>

to default `PropertyGroup` in `*.csproj` files.

Ref: [https://github.com/sall/vixen/pull/173](https://github.com/sall/vixen/pull/173)