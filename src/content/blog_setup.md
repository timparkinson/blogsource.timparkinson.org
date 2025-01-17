+++
date = '2024-11-24T05:25:08Z'
draft = false
title = 'Blog setup'
description = 'How this blog is built'
tags = ["blog"]
+++

## Hugo

I like Static Site Generators (SSGs) for a blog, and I've used [Hugo](https://gohugo.io) in the past, so I've built this using that. I did a bit of searching to see if there was anything equivalent in the .Net space, but couldn't really find much other than [docfx](https://dotnet.github.io/docfx/), which is a bit documentation focused. 

If anything, Hugo is actually too powerful and comprehensive a tool. Having not used it in a while it took me a little bit of poking and prodding to get going with it. 

## Theme

I spent a few hours on a Sunday morning looking at, and wrestling with, Hugo themes. I found that quite a lot of them are very heavyweight and feature-rich, which is overblown for the very simple requirement that I have. I ended up hand-cranking layout files using [simple.css](https://simplecss.org), this is small, opinionated and quick to get going with and this suits my use case.

That's not to say that I won't ~pinch~ add some of the features found in Hugo themes, and I may even abstract my setup out into a theme in the future.

The only change I've made so far, in a `custom.css` file, is to add a style for a non-bulleted list.

```css
ul.no-bullets {
    list-style-type: none; /* Remove bullets */
    padding: 0; /* Remove padding */
    margin: 0; /* Remove margins */
  }
```

## Hosting

I've had a [Dreamhost](https://www.dreamhost.com/) hosting account for a __long__ time. I don't think that a static blog site that nobody is going to read needs to be any fancier than that to be honest. 

I use [Cloudflare](https://www.cloudflare.com/) to host the DNS zone for `timparkinson.org` and act as a CDN. I let Cloudflare handle the 'Edge' certificates automatically - i.e. those that are presented to the browser by Cloudflare's magic proxying. I've installed a Cloudflare Origin certificate on the Dreamhost site which allows for end to end encryption. I did have to switch the TLS settings to `Full (strict)` within Cloudflare otherwise I got stuck in a redirect loop.

I'm using [Github Actions](https://github.com/features/actions) to build and publish the site, and will detail that setup at a later time.   
