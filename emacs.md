<!--
// Copyright 2017 The Fuchsia Authors. All rights reserved.
-->
# Notes on using Emacs with Fuchsia

## FIDL syntax highlighting

Emacs comes with builtin support for IDL files,
so this is a good first pass. In your .emacs file add:

```
(add-to-list 'auto-mode-alist '("\\.fidl\\'" . idl-mode))
```
