# Emacs Bug #81723 — Fix Package

Prepared by Alexander Leonard, 2026-09-04.

## Bug

Assertion crash in `src/syntax.c` (`SINGLE_BYTE_CHAR_P`) triggered
during sexp scanning in buffers with mixed encodings.

Minimal reproducer (Emacs built with `--enable-checking`):

```elisp
(with-temp-buffer (set-buffer-multibyte nil)
  (syntax-class #x3042))
;; => character.h:150: Emacs fatal error: assertion failed: SINGLE_BYTE_CHAR_P (c)
```

Full report: [BUG-REPORT.md](BUG-REPORT.md)

## Contents

1. `syntax-c.patch`
   C-level guard: multibyte characters are resolved through their syntax
   descriptor before any single-byte table lookup, so the assertion site
   is no longer reachable with invalid state.

2. `elisp-fontify-fix.el`
   Session-level workaround for the input lag exposed by
   `elisp-fontify-semantically`:
   - trust gating (safe-local buffers only),
   - per-buffer decision cache,
   - automatic opt-out above 50KB.

   *(not included — this file was not present in the filed fix
   materials, so it is not shipped here; the C patch above is the
   complete fix as filed)*

## Usage

Apply the patch against a current Emacs tree.
