# Bug report (as filed upstream)

Using an Emacs built with '--enable-checking', calling `syntax-class'
with a multibyte character while the current buffer is unibyte triggers
an assertion:
>
$ emacs -Q --eval '(with-temp-buffer (set-buffer-multibyte nil)
(syntax-class #x3042))'
character.h:150: Emacs fatal error: assertion failed: SINGLE_BYTE_CHAR_P (c)
Fatal error 6: Aborted
>
AFAIU, it is not meaningful to request syntax information for multibyte
characters in a unibyte buffer, since the buffer's syntax table is
only defined for single-byte characters in that situation.

Assuming I followed the remaining code path correctly,
org-mode isn't actually attempting to inspect syntax "inside" any
particular buffer:

When loaded, org-mode builds a separate syntax table and temporarily
makes it the active syntax table so it can conditionally update several
entries in it (see
https://orgmode.org/worg/org-syntax.html).

The crash I encountered happened during `package-install`, which first
called `package--with-response-buffer-1` (leaving the current buffer in
unibyte mode), followed by `package--reload-previously-loaded`, which
loaded org-mode while the temporary package buffer was still current.

(Let me know if you want a backtrace or more reproduction details:
I may have made the example slightly too minimal.)

org-mode really isn't interested in the syntax of a multibyte
character "inside" the buffer previously created by package.el: I think
it is only doing this because `char-syntax` is the most straightforward
way to query the syntax table it is preparing. I _think_
(but could easily have missed something, since I hadn't worked with
syntax tables before debugging this...) that set-syntax-table is valid
for temporarily installing a syntax table intended for a later
multibyte buffer while the current buffer happens to be unibyte.

org-mode has since added a `with-temp-buffer` around the
`with-syntax-table` call, so that should work with the proposed change.
There may nevertheless be other packages performing a similar operation.

Stefan, is my interpretation correct? If so, does the patch below
seem like the right solution?
>
If not, what behavior would be preferable? Should we silently
return a default syntax value instead?
>
