---
title: "Debugging ASP.NET MVC 2 Source Code"
date: 2010-06-27 14:53:42Z
draft: false
aliases:
- /post/debugging-asp-net-mvc-2-source-code/
categories:
- Patterns
- MVC
---
I'm trying to write some templates for MVC 2 and one of the most useful things I've found is being able to step into the source code and see how it is actually interpreting what you've written.

It's easy enough to acquire the [source](http://aspnet.codeplex.com/releases/view/41742), but what you might not have done before is set up a symbol server. A symbol server is a location that the Windows and Visual Studio debugging tools can use to obtain pdb files, so that you can debug almost anything, including drivers and the operating system!

## Setup Symbol Server

1. Create a directory, preferably in a shared location if there's more than one of you as this stuff can get quite big. 
1. Change the directory to allow compression as pdb's will typically compress by > 50%. 
1. Add a new system environment variable `_NT_SYMBOL_PATH` and set it to SRV**<Shared Directory>**http://referencesource.microsoft.com/symbols;http://msdl.microsoft.com/download/symbols

![](/img/vssrc2_2.png) 

## Enable Debug Symbols in Visual Studio

1. Install the [debugging tools for Windows](http://www.microsoft.com/whdc/devtools/debugging/default.mspx) and run Start Menu|Programs|Microsoft Windows SDK 7.1|Visual Studio Registration|Windows SDK Configuration Tool; this allows Visual Studio to use the symbols.
1. Run Visual Studio and go to Tools -> Option -> Debugging -> General 
1. Uncheck **Enable Just My Code** 
1. Check **Enable .NET Framework source stepping** ![](/img/vssrc1_2.png) 
1. Change to Symbols 
1. Update "Cache symbols from symbol servers to this directory" to point at your shared symbol directory, then hit "Load symbols from Microsoft symbol servers", this will take a while and chew up about 111Mb (~48Mb compressed) 
1. Put a break point in one of you controllers and run through to that point. 
1. Pick a stack frame from System.Web.Mvc from Call Stack and select Load Symbols if it is not disabled and then Go To Source Code, picking the location you have put MVC 2 ![](/img/vssrc3_2.png) 

One other interesting angle on this is [Symbol Source](http://www.symbolsource.org) who are providing symbol/source service for a bunch of open source projects including NHibernate, Castle and MVC Contrib