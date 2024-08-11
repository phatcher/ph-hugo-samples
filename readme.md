# Introduction

On first cloning run

```git submodule update --init```

Sometime this fails on first run in which case you have to re-add the submodule explicitly

```git submodule add https://github.com/vimux/mainroad themes/mainroad```

See the [SO answer](https://stackoverflow.com/questions/3336995/git-will-not-init-sync-update-new-submodules) has more details.

to acquire the themes

# Running the blog locally

The following command will build the blog and publish locally all content including draft and future dated

```hugo server --port 1320 --buildDrafts --buildFuture --bind 0.0.0.0 -D```