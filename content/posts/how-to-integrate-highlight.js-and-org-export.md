---
title: "Making highlight.js work with org-export"
description: "A blog post about making highlight.js work with org-export."
date: "2024-01-01"
tags: ["blog"]
---



Org-mode can export to HTML, PDF and LaTeX, but its HTML export doesn't highlight code blocks, which makes code hard to read. [highlight.js](https://highlightjs.org/) fixes that. It's a JavaScript syntax highlighter that works on almost any markup, doesn't depend on other frameworks, and detects the language on its own.

This post assumes you've already set up `org-publish-project-alist` to export your `.org` files to `.html`. We'll change that variable to load highlight.js.

## Modifying org-publish-project-alist

Your `org-publish-project-alist` probably looks something like this:

```elisp
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

To load highlight.js, add these two options:

```elisp
             :html-preamble "
<link rel=\"stylesheet\" href=\"https://unpkg.com/highlightjs@9.16.2/styles/obsidian.css\">
<script src=\"https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.7.0/highlight.min.js\"></script>
"
             :html-postamble "
<script>hljs.highlightAll();</script>
"
```

`:html-preamble` adds HTML to the top of every page. The first line loads the highlight.js stylesheet and the second loads the script. `:html-postamble` adds HTML to the bottom of the page, where we call `hljs.highlightAll();`.

## Modifying org-html-src-block

If you stop here, your code blocks still won't be highlighted. highlight.js looks for code inside a `<code>` tag within a `<pre>`, like this:

```html
<pre>
<code>
  Code goes here
</code>
</pre>
```

Org's HTML export writes a plain `<pre class="src src-lang">` with no `<code>` tag, so highlight.js skips it. This snippet rewrites the output into the format highlight.js wants. I didn't write it. I found it in a Stack Overflow post but can't find the link anymore.

```elisp
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

`advice-add` runs `my/org-html-src-block` on the output of `org-html-src-block` every time org exports a source block.
