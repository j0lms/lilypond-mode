This repo contains the Emacs `lilypond-mode` files from the upstream version of LilyPond.

This is meant to be used with elpaca without needing to clone the entire LilyPond repo or use Python to create `lilypond-words.el`, like so:

```elisp
(use-package lilypond-mode
  :ensure (lilypond-mode
	 :type git
	 :host github
	 :repo "j0lms/lilypond-mode"
     :files ("*.el"))
  :mode ("\\.ly$" . lilypond-mode)
  :commands (lilypond-mode))
```

Additionally, `lilypond-words.el`, which is generated from the `/scripts/build/lilypond-words.py` script, was causing an EOF error.

The original `lilypond-mode.el` attempted to parse the entire `lilypond-words.el` file using the `read` function.

This change was made to resolve the parsing issue:

```emacs-lisp
;; Original code:
(if (and (eq (length (lilypond-add-dictionary-word ())) 1)
	 (not (eq (lilypond-words-filename) nil)))
    (progn
      (setq b (find-file-noselect (lilypond-words-filename) t t))
      (setq m (set-marker (make-marker) 1 (get-buffer b)))
      (setq i 1)
      (while (> (get-buffer-size b) (marker-position m))
	(setq i (+ i 1))
	;; This line caused the "end-of-file" error by trying to read plain text as Lisp.
	(setq copy (copy-alist (list (eval (symbol-name (read m))))))
	(setcdr copy i)
	(lilypond-add-dictionary-word (list copy)))
      (kill-buffer b)))

;; Corrected code:
(if (and (eq (length (lilypond-add-dictionary-word ())) 1)
	 (not (eq (lilypond-words-filename) nil)))
    (let* ((filename (lilypond-words-filename))
           ;; Read the entire file content as a string.
           (file-content (with-temp-buffer
                           (insert-file-contents filename)
                           (buffer-string)))
           ;; Split the string into words by newlines.
           (words (split-string file-content "\(
\|\r\n\)" t)))
      ;; Add each non-empty word to the dictionary.
      (dolist (word words)
        (if (not (string-empty-p word))
            (lilypond-add-dictionary-word (list (cons word 1)))))))
```




