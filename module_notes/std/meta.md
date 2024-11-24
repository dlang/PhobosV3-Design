# Overview

This provides information on the symbols in std.meta with regards to what's
happening to them in Phobos v3. Either it indicates which Phobos v3 symbol
should be used instead, or it explains why that symbol hasn't been ported to
Phobos v3.

[`phobos.sys.meta`](../phobos/sys/meta.md) is the Phobos v3 replacement for
std.meta, so in general, that's where the symbols from std.meta have been
ported to, but not all are getting ported, and some of them have had their
names and/or functionality changed in the process.

---
# Symbols

## Alias

* See [`phobos.sys.meta.Alias`](../phobos/sys/meta.md#alias).

## AliasSeq

* See [`phobos.sys.meta.AliasSeq`](../phobos/sys/meta.md#aliasseq).

## aliasSeqOf

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's barely used in Phobos v2.
* Also, even if we do decide to port it over, the current implementation
  requires std.array.array, which has not yet been ported to Phobos v3.

## allSatisfy

* See [`phobos.sys.meta.all`](../phobos/sys/meta.md#all).

## anySatisfy

* See [`phobos.sys.meta.any`](../phobos/sys/meta.md#any).

## ApplyLeft

* See [`phobos.sys.meta.ApplyLeft`](../phobos/sys/meta.md#applyleft).

## ApplyRight

* See [`phobos.sys.meta.ApplyRight`](../phobos/sys/meta.md#applyright).

## DerivedToFront

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's not used anywhere in Phobos v2
  outside of std.meta.

## Erase

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's currently used in only one place in
  Phobos v2 outside of std.meta.
* It's arguably a trivial wrapper around [`staticIndexOf`](#staticindexof),
  which would arguably still be useful if it were needed frequently, but it
  isn't.
* If we do decide to add it, it will need to be reworked to take a template
  predicate (which [`phobos.sys.meta.indexOf`](../phobos/sys/meta.md#indexof)
  would require, unlike [`staticIndexOf`](#staticindexof)). So, it would
  essentially become a version of [`Filter`](../phobos/sys/meta.md#filter) that
  stopped after matching a single element (or after encountering an element
  which didn't match). So, if we _do_ decide to add something along these lines
  to Phobos v3, we should probably add something like `FilterUntil` rather than
  actually port over `Erase`. Ultimately though, it's probably better to have
  code that wants `Erase` to just use
  [`indexOf`](../phobos/sys/meta.md#indexof), particularly since it does not
  appear to be a common use case.

## EraseAll

* There are no plans to port it over to Phobos v3.
* [`Filter`](../phobos/sys/meta.md#filter) can be used instead, since `EraseAll` is
  essentially [`Filter`](../phobos/sys/meta.md#filter) with a predicate for
  comparing the elements which is supplied by std.meta (and it's a predicate
  that we really don't want to continue to use given how quirky and error-prone
  it is).

## Filter

* See [`phobos.sys.meta.Filter`](../phobos/sys/meta.md#filter).

## Instantiate

* See [`phobos.sys.meta.Instantiate`](../phobos/sys/meta.md#instantiate).

## MostDerived

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's currently used in only one place in
  Phobos v2 outside of std.meta.

## NoDuplicates

* See [`phobos.sys.meta.Unique`](../phobos/sys/meta.md#unique).

## Repeat

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's not used anywhere in Phobos v2
  outside of std.meta.

## Replace

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's currently used in only one place in
  Phobos v2 outside of std.meta.

## ReplaceAll

* There are no plans to port it over to Phobos v3.
* Its usefulness is questionable, and it's not used anywhere in Phobos v2
  outside of std.meta.

## Reverse

* See [`phobos.sys.meta.Reverse`](../phobos/sys/meta.md#reverse).

## staticIndexOf

* See [`phobos.sys.meta.indexOf`](../phobos/sys/meta.md#indexof).

## staticMap

* See [`phobos.sys.meta.Map`](../phobos/sys/meta.md#map).

## staticIsSorted

* There are no plans to port it over to Phobos v3.
* It does seem like it _might_ be useful under some set of circumstances, but
  it's not actually used anywhere in Phobos v2 outside of std.meta. So, it's
  certainly not something that seems like it fundamentally shouldn't be
  included, but it also doesn't seem to be particularly useful in practice.

## staticSort

* There are no plans to port it over to Phobos v3.
* It does seem like it _might_ be useful under some set of circumstances, but
  it's not actually used anywhere in Phobos v2 outside of std.meta. So, it's
  certainly not something that seems like it fundamentally shouldn't be
  included, but it also doesn't seem to be particularly useful in practice.

## Stride

* See [`phobos.sys.meta.Stride`](../phobos/sys/meta.md#stride).

## templateAnd

* See [`phobos.sys.meta.And`](../phobos/sys/meta.md#and).

## templateNot

* See [`phobos.sys.meta.Not`](../phobos/sys/meta.md#not).

## templateOr

* See [`phobos.sys.meta.Or`](../phobos/sys/meta.md#or).
