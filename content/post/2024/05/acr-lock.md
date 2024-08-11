---
title: "Locking manifests in Azure Container Registry"
date: 2024-05-20 09:00:00Z
draft: false
categories:
- DevOps
- Azure
- ACR
---
So a consequence of wanting to purge images from your container registry is sometimes you need to keep some of them.

One example is that in regulated industries you must be able to produce software that was deployed to production for audit/legal purposes, so if you are purging images that are over 3 months old you might have an issue.

You can update an image so that it can't be overridden or deleted via the `az acr repository update` command e.g.

```
az acr repository update -n myregistry \
    --image myimage@sha256:1234abcde \
    --delete-enabled false --write-enabled false
```

This will protect your image, but if this image is broken, e.g. critical CVEs, you might need to keep it for audit purposes, but not let it be deployed, so we can prevent pulling via this:

```
az acr repository update -n myregistry \
    --image myimage@sha256:1234abcde \
    --read-enabled false
```

In our classic release pipelines, we have variables for the image that will be deployed, so on promotion to the production ACR, and deployment to the production environment, we can lock it with an Azure CLI task like this...

```
# Get the manifest 
$manifest = az acr repository show -n $(registryName) --image $(imageName):$(imageTag) | ConvertFrom-Json

# Pin it in the ACR
$digest = "$(imageName)@$($manifest.digest)"
az acr repository update --name $(registryName) --image ($digest) --delete-enabled false --write-enabled false
```

This ensures we lock the underlying manifest rather than just the tag we are deploying and prevents it from being removed by our purge routines.