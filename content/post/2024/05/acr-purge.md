---
title: "Purging manifests from Azure Container Registry"
date: 2024-05-19 09:00:00Z
draft: false
categories:
- DevOps
- Azure
- ACR
---
One housekeeping task if you are producing docker images is clearing your container registry of old images.

I've been particularly bad at this and had also made some mistakes such as publishing PR images and images produced via our auto-patching runs, leading to the ACR size being >1.2Tb (ouch!).

There are some features introduced by [acr auto purge](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auto-purge), but I found an issue since my images typically have three tags...

1. Build number produced by Azure DevOps, needed as release pipelines can't pass the version
1. The git SHA1, easy tracing back to the source code rather than just the build
1. Version number produced by gitversion (also on the source as a tag)

When I tried to remove an image via a tag, it removed the tag but not the manifest due to the other tags referencing it. The [documentation](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-delete#delete-by-tag) says that this should work, but it doesn't appear to.

So I wrote some helper functions to allow me to remove a manifest even if other tags reference it.

The top level function takes a few interesting parameters...

* **repos**: Single or list of repositories, if empty will retrieve all repositories from the ACR.
* **filter**: Filter to match on the tag
* **age**: A simple go style age, e.g 12h(ours), 2d(ays), 3w(eeks)
* **dryRun**: Whether we should really run the command, allows previewing the actions to be taken.

One little bit of protection it that it stops you using an empty filter with less than 30 days, avoids those sorts of embarrassing errors where you delete *all* the images!

One thing, if you have a large number of repositories/images, execute this in a [Azure Cloud Shell](https://learn.microsoft.com/en-us/azure/cloud-shell/using-the-shell-window) as this speeds up the process a lot, as does using different sessions in parallel.

The result in my case is we saved over 1Tb in space, and can probably reduce it more but it will need more detailed analysis.

Here's the code

{{< gist phatcher f4f007b8d29f1b54484c3e7bb7d4d7fc "acr.psm1" >}}
