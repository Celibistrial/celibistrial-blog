---
title: "Making highlight.js work with org-export"
description: "A blog post about making highlight.js work with org-export."
createdAt: "2026-04-04"
tags: ["blog"]
---


# Making highlight.js work with org-export

> Highlight.js is a syntax highlighter written in JavaScript. It works in the browser as well as on the server. It can work with pretty much any markup, doesn’t depend on any other frameworks, and has automatic language detection.

## Intro

Org-mode's most notable features is the ability to export content to different formats, including HTML, PDF, and LaTeX, but it has its limitations including the lack of syntax highlighting, which can make exported code blocks challenging to read.

However, integrating [highlight.js](https://highlightjs.org/) can solve this problem by adding colour and emphasis to code blocks, making them more accessible and visually appealing.

## Getting started

I assume that you have already set up the org-publish-project-alist to export your `.org` files to `.html` files

We will be modifying the **org-publish-project-alist** variable to bundle in highlight.js

## Modifying org-publish-project-alist

Your org-publish-project-alist should look something like this

```
(setq org-publish-project-alist
      (list
       (list "celibistrial-website"
             :recursive t
             :auto-sitemap t
             :base-directory "~/org/celibistrial-website/content"
             :publishing-function 'org-html-publish-to-html
             :publishing-directory "~/org/celibistrial-website/public"
             :with-author nil
             :footnote-section-p t
             :html-footnotes-section t
             :html-doctype "<!doctype html>"
            :time-stamp-file nil)))
```

To bundle highlight.js simply add these 2 lines

```
             :html-preamble "
<link rel=\"stylesheet\" href=\"https://unpkg.com/highlightjs@9.16.2/styles/obsidian.css\">
<script src=\"https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.7.0/highlight.min.js\"></script>
"
             :html-postamble "
<script>hljs.highlightAll();</script>
"
```

The `:html-preamble` adds the HTML to the top of the document, the first line pulls in the CSS files required for highlight.js and the second line pulls the script

Then at the end of the document (using `:html-postamble`) we call `hljs.highlightAll();`

## Modifying org-html-src-block

You may have tried using the above steps and are confused as to why highlight.js is not highlighting your code blocks.

Well highlight.js expects code blocks in a specific format

```
<pre>
<code>
  Code goes here
</code>
</pre>
```

Highlight.js expects code blocks in the format specified above but by default org-export uses a different format

Changing the format is simple

```
;; I did not write this , i found it from a stackoverflow post but i am unable to find a link to it
 (defun my/org-html-src-block (html)
  "Modify the output of org-html-src-block for highlight.js"
  (replace-regexp-in-string
   "</pre>" "</code></pre>"
   (replace-regexp-in-string
    "<pre class=\"src src-\\(.*\\)\">"
    "<pre><code class=\"\\1\">"
    html)))

(advice-add 'org-html-src-block :filter-return #'my/org-html-src-block)
```

We can use advice-add to modify the output of org-html-src-block so that it is compatible with highlight.js
