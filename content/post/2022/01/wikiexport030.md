---
title: "Wikiexport 0.3"
date: 2022-01-23 17:00:00Z
draft: false
categories:
- DevOps
- Azure
---
Just pushed a new version of [WikiExport](https://github.com/phatcher/wikiexport) with a couple of useful features

* Change `titleFormat` to have explicit `{project}`/`{title}` macros 
* Support for .attachments in non-root folder - provided by [@ricfre](https://github.com/ricfre/wikiexport)

The first is just a simplification since it's more intentional to use named values rather than `{0}` etc, and we can also automatically determine `projectInTitle` from the `titleFormat`.

The second one allows support for [code wikis](https://docs.microsoft.com/en-us/azure/devops/project/wiki/publish-repo-to-wiki?view=azure-devops&tabs=browser) and the image path not being in the root of the repository e.g. the repository may look a bit like this...

```
$
  src
    ...
  wiki
    .attachments
      cat.png
    .order
    doc.md
```

For this to work as a wiki, the image references need to be like this `/wiki/.attachments/cat.png` 

It is possible to have the `.attachments` folder in

* Root of repostory
* Root of code wiki
* Any sub-folder of the code wiki folder

and the tool should still convert the image references correctly on export.

There are also some fixes to use the slash path delimiter (`/`) in all cases as some markdown processors get confused if you use backslash (`\`).