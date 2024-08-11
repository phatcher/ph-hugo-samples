---
title: "Four things you need to know about Recaptcha"
date: 2010-06-26 10:02:40Z
aliases:
- /post/four-things-you-need-to-know-about-recaptcha/
draft: false
---
Working on a project where we needed a (slight) proof of humaness, and we opted to use Recaptcha as it has a fairly simple integration with .NET.

Was fine until we wanted to use custom styling to tie in with the site which is when the fun started! A few hours later, after downloading the project's sample app, I found some things out that I thought I'd share with you...

1. The Recaptcha control must be positioned after the `recaptcha_image` div and `recaptcha_response_field`
1. The killer: your recaptcha_response_field **must** have a name attribute of `recaptcha_response_field` as well as its id - otherwise Recaptcha won't see it.
1. You can specify your public/private key in the web.config AppSettings rather than against the control; `RecaptchaPublicKey` and `RecaptchaPrivateKey` respectively.
1. You can turn off Recaptcha validation either by setting the `SkipRecaptcha` property of the control or `RecaptchaSkipValidation`.

The first two are the most important to getting Recaptcha to work with custom styling; I know the properties are documented on the control, but I managed to miss them for a couple of hours so I thought others might do the same.