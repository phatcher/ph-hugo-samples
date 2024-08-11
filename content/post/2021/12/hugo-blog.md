---
title: "Moved Blog to Hugo"
date: 2021-12-31 14:49:50Z
draft: false
---
I need to resume my blog to talk about some projects I've been working on and I've migrated over to using [Hugo](https://gohugo.io/) as the blogging platform from my old MiniBlog based solution. The main reasons are ease of writing since the content typically has some code in it, and with Hugo I can write in Markdown. Went relatively easily since I didn't have that many posts to fix but did find some that had gone missing :smile:.

Under the hood it uses private github repo and an [Azure Static Web Site](https://docs.microsoft.com/en-us/azure/static-web-apps/publish-hugo).

Changed a couple of things to make life easier:
* Scheduled publication
* External links open in new tab
* Disclaimer widget

Now that I've made the process simple, hopefully this means I can write more next year as I have some things that I've been working on that you hopefully will find interesting.

## Scheduled Publication

I'd like to do my writing in batches when I have time and then publish them out over time. Hugo will obey the `date` property of the post frontmatter during generation, so the question is how to make them appear on the appropriate day?

The static web site is published via a GitHub Action held in the .github folder of your Hugo project and looks like this...

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - master
...
```

This will re-build the site when
* Changes are pushed to `master`
* Pull requests are actioned

What we need is to extend this with a schedule something like this...

```yaml
name: Azure Static Web Apps CI/CD

on:
  schedule:
  - cron: '0 9 * * *'
...
```
This is a standard cron specification, so in addition to the other triggers it will schedule a build at 09:00 every day, so I need to  date the post with a time earlier than that otherwise it won't appear until the next day.

## External links

For internal links you want to stay within the site, but for externals I prefer them to open in a new tab. The technique for this I picked this up from [Agrim Prassad's blog](https://agrimprasad.com/post/hugo-goldmark-markdown/) and it basically is putting a new link render in the layout folder i.e.

```
layouts
└── _default
    └── _markup
        └── render-link.html
```

## Disclaimer widget

As this is a personal blog I need to make it clear that the opinions etc are mine and that I don't speak for my employer.

I decided to do this with an extension of the current widget idea in [MainRoad](https://github.com/Vimux/Mainroad) so I can put the content easily in the right sidebar.

So I need three things...

1. The disclaimer content
1. Enabling the disclaimer in the widget list
1. The partial template to render the disclaimer

The first two can be solved by setting parameters in the `config.toml`

```
[Params]
  description = "Technology, business and other musings"
  copyright = "Paul Hatcher"
  disclaimer = "The opinions expressed herein are my own personal opinions and do not represent my employer’s view in any way."
  ...

[Params.sidebar]
  home = "right" # Configure layout for home page
  list = "right"  # Configure layout for list pages
  single = "right" # Configure layout for single pages
  # Enable widgets in given order
  widgets = ["search", "recent", "categories", "taglist", "social", "disclaimer", "languages"]
  # alternatively "ddg-search" can be used, to search via DuckDuckGo
  # widgets = ["ddg-search", "recent", "categories", "taglist", "social", "languages"]
```
By separating in this way I have the option to change where the disclaimer goes e.g. the footer without by adjusting the footer partial and removing `disclaimer` from the list of widgets.

To avoid customizing the theme, I put the new template in my own layout folder, a similar process to the previous `render-link.html` idea...
```
layouts
└── partials
    └── widgets
        └── disclaimer.html
```

And the template looks like this...
```html
{{ with $.Site.Params.disclaimer }}
<div class="widget-disclaimer widget">
	<h4 class="widget__title">Disclaimer</h4>
	<div class="widget__content">{{ $.Site.Params.disclaimer }}</div> 
</div>
{{ end}}
```
The conditional content means that it won't render if there is no disclaimer content.