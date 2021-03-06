---
navhome: /docs/
next: true
sort: 2
title: |= "bartis"
---

# `|= "bartis"` 

`[%brts p=model q=hoon]`: form a gate, a dry one-armed core with sample.

## Expands to

```
=|  p
|%  ++  $  q
--
```

## Syntax

Regular: *2-fixed*.

## Discussion

A gate is a core with one arm named `$`, so, just as with 
[`|-` ("barhep")](../hep), we can recurse back into it with `$()`.

> `$()` expands to `%=($)` (["centis"](../../cen/tis)), accepting 
> a *jogging* body containing a list of changes to the subject.

## Examples

A trivial gate:

```
~zod:dojo> =foo |=(a=@ +(a))
~zod:dojo> (foo 20)
21
```

A slightly less trivial gate:

```
~zod:dojo> =foo  |=  [a=@ b=@]
                 (add a b)
~zod:dojo> (foo 20 400)
420
```
