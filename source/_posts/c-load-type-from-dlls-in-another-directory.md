---
title: C# load Type from DLLs in another directory
tags:
  - DLL
id: '19'
categories:
  - - C#
  - - c
    - Vixen
date: 2017-06-05 21:50:39
---

To search DLLs in another directory for

Type.GetType()

e.g. (From `Modules/Controller/TCPLinky.dll` loaded using `Assembly.LoadFrom`)

Type.GetType("VixenModules.Output.TCPLinky.TCPLinkyData, TCPLinky")

Add following to app.config:

<configuration>
   <runtime>
      <assemblyBinding xmlns=“urn:schemas-microsoft-com:asm.v1”>
         <probing privatePath=“v1” />
      </assemblyBinding>
   </runtime>
</configuration>

Ref: [https://blogs.msdn.microsoft.com/.../load-contexts-and-type-gettype/](https://blogs.msdn.microsoft.com/thottams/2007/05/11/load-contexts-and-type-gettype/)