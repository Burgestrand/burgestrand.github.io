---
title: My recent quarter with Swift
date: 2017-01-20T17:03:10+01:00
categories: []
---

I've been doing a lot of Swift lately, roughly since early September. I do like
types, and generics, and how the compiler tells me when I make mistakes. For the
record, I still very much like Ruby but for different reasons. I remain a
polyglot.

Like any C-like language it's rather easy to pick up — it has types, it has
classes, it has generics. My overall feeling of the language is pleasant, even
though you certainly feel that it's not quite done yet. Basic things are still
missing, for example this code won't compile.

``` swift
class LinkedList {
  class Node<T> {
  }
}
```
