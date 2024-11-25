+++
date = '2024-11-25T14:56:45Z'
draft = false
title = 'Simple shortcodes'
description = 'Adding additional styling shortcuts'
tags = ["blog"]
+++

## Introduction

The [simple.css](https://simplecss.org) offers a selection of neatly formatted elements - as shown on the [demo](https://simplecss.org/demo) page. Some of these aren't available as markdown elements though, so aren't directly usable. It is possible to turn on Hugo's ability to render HTML tags within the markdown source, however it's considered [^tangent] [^paks] more idiomatic to use [shortcodes](https://gohugo.io/content-management/shortcodes/).

Creating shortcodes rather than using 'unsafe' HTML does mean that changes at a later time are easier. Modifications may be made in one place, the shortcode definition, rather than lots of content files.

## Creating shortcodes

Shortcodes are just more HTML templates, placed in the `layouts/shortcodes` directory. There are some example custom shortcodes of varying complexity within the [Hugo documentation](https://gohugo.io/templates/shortcode/#custom-shortcode-examples). I've created some shortcodes implementing some of the elements that simple.css provides - but not all of them, yet.

In the example code below I've had to escape the shortcodes to stop them rendering [^liatas].

### Linkbutton

`layouts/shortcodes/linkbutton.html`

```html
<a class="button" href="{{ index .Params 0 }}">{{ index .Params 1 }}</a>
```

```
{{</* linkbutton "https://blog.timparkinson.org" "Worst blog ever" */>}}
```
{{< linkbutton "https://blog.timparkinson.org" "Worst blog ever" >}}

### Mark

`layouts/shortcodes/mark.html`

```html
<mark>{{ .Inner }}</mark>
```

```
{{</* mark */>}}Highlighted{{</* /mark */>}} Text.
```
{{< mark >}}Highlighted{{< /mark >}} Text.

### Kbd

`layouts/shortcodes/kbd.html`

```html
<kbd>{{ index .Params 0 }}</kbd>
```

```
Press {{</* kbd "Alt+F4" */>}} to active the destruct sequence.
```
Press {{< kbd "Alt+F4" >}} to active the destruct sequence.

### Notice

`layouts/shortcodes/notice.html`

```html
<p class="notice">{{ .Inner | markdownify}}</p>
```

```
{{</* notice */>}}It's important to draw your attention to this thing.{{</* /notice */>}}.
```
{{< notice >}}It's important to draw your attention to this thing.{{< /notice >}}

### Aside

```html
<aside>{{ .Inner }}</aside>
```

```
{{</* aside */>}}A small anecdote about something referenced in the text proper. {{</* /aside */>}}

A very important discussion about something very serious. You do not want to distract the reader from the main thrust of your point by including trivialities here.
```

{{< aside >}}A small anecdote about something referenced in the text proper. {{< /aside >}}

A very important discussion about something very serious. You do not want to distract the reader from the main thrust of your point by including trivialities here.

## Section

```html
<section>{{ .Inner | markdownify }}</section>
```

```
{{</* section */>}}

### Title

Some section content

{{</* /section */>}}

```

{{< section >}}

### Title

Some section content

{{< /section >}}


[^tangent]: https://tangenttechnologies.ca/blog/hugo-shortcodes/
[^paks]: https://pakstech.com/blog/hugo-shortcodes/
[^liatas]: https://liatas.com/posts/escaping-hugo-shortcodes/
