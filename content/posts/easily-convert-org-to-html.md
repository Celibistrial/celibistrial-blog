---
title: "How to easily convert org to html"
description: "A blog post about how to easily convert org to html."
date: "2024-01-01"
tags: ["blog"]
---

Here's a snippet that lets you pick a CSS theme whenever you export an org file to HTML.

1.  Put your CSS files in `~/.emacs.d/org-css/`, or change `org-theme-css-dir` to wherever you keep them.
2.  Run `toggle-org-custom-inline-style` in an org buffer that's visiting a file.
3.  Export to HTML with `C-c C-e`. It should now ask you which theme to use.

```elisp
;; put your css files there
(defvar org-theme-css-dir "~/.emacs.d/org-css/")

(defun toggle-org-custom-inline-style ()
  (interactive)
  (let ((hook 'org-export-before-parsing-hook)
        (fun 'set-org-html-style))
    (if (memq fun (eval hook))
        (progn
          (remove-hook hook fun 'buffer-local)
          (message "Removed %s from %s" (symbol-name fun) (symbol-name hook)))
      (add-hook hook fun nil 'buffer-local)
      (message "Added %s to %s" (symbol-name fun) (symbol-name hook)))))

(defun org-theme ()
  (interactive)
  (let* ((cssdir org-theme-css-dir)
         (css-choices (directory-files cssdir nil ".css$"))
         (css (completing-read "theme: " css-choices nil t)))
    (concat cssdir css)))

(defun set-org-html-style (&optional backend)
  (interactive)
  (when (or (null backend) (eq backend 'html))
    (let ((f (or (and (boundp 'org-theme-css) org-theme-css) (org-theme))))
      (if (file-exists-p f)
          (progn
            (set (make-local-variable 'org-theme-css) f)
            (set (make-local-variable 'org-html-head)
                 (with-temp-buffer
                   (insert "<style type=\"text/css\">\n<!--/*--><![CDATA[/*><!--*/\n")
                   (insert-file-contents f)
                   (goto-char (point-max))
                   (insert "\n/*]]>*/-->\n</style>\n")
                   (buffer-string)))
            (set (make-local-variable 'org-html-head-include-default-style)
                 nil)
            (message "Set custom style from %s" f))
        (message "Custom header file %s doesnt exist")))))
```

I didn't write this. I found it in a comment by [u/aaptel](https://www.reddit.com/r/emacs/comments/3pvbag/is_there_a_collection_of_css_styles_for_org/) and wrote this post so it's easier for other people to find.
