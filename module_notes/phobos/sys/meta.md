# Overview

This provides some notes on the design of phobos.sys.meta.

phobos.sys.meta is a port of Phobos v2's std.meta with changes where
appropriate. The basic goal of the module is the same in that it's supposed to
be providing templates that do metaprogramming on `AliasSeq`s - essentially a
std.algorithm for `AliasSeq`s instead of ranges.

phobos.sys.meta has fewer symbols than std.meta primarily because it's best to
use CTFE rather than template metaprogramming where possible and because a
number of the templates in std.meta are of questionable utility. Clearly,
_someone_ thought that they would be useful, or they wouldn't have been added,
but Phobos either doesn't use them at all or barely uses them, making it
questionable that we want them in the standard library. So, while we may add
some of those symbols later, for now at least, if a symbol doesn't seem like
it's generally useful, it's not being ported.

A major functionality change between std.meta and phobos.sys.meta is that
various templates in std.meta compare the elements in an `AliasSeq` - typically
using a private template which tries to do a bunch of different kinds of
comparisons with the behavior changing depending on whether the element can be
compared with `==`, an `is` expression, or `__traits(isSame, ...)`. This has
proven to be incredibly error-prone with some weird and unexpected behaviors
(e.g. whether a no-argument function is treated as a value or as a symbol
depends on what it's being compared against as well as which side of the
comparison it's on). And in general, operating on an `AliasSeq` that's a
mixture of types, values, and symbols is just a mess. So, instead of having
phobos.sys.meta's templates provide any sort of comparison, they instead now
take template predicates (typically a trait from phobos.sys.traits) so that the
programmer can decide how the comparison is done. So, ultimately, the same
functionality is provided, but it's under the control of the programmer, making
it much more flexible and less error-prone.

Another functionality change is that std.meta is inconsistent about whether a
template that does a comparison short-circuits evaluation, whereas
phobos.sys.meta's templates do not short-circuit evaluation - both for the sake
of consistency, and because it allows for more efficient implementations. Also,
given that some of the elments in an `AliasSeq` might not even compile if they
are evaluated (e.g. because the predicate assumes a type, and an element is a
value), it's arguably better to evaluate all of the elements and catch such
bugs.

With regards to the name changes, to better follow the official style guide,
any templates that evaluate to a value are camelCased, any templates which
evaluate to a type are PascalCased, and any templates which evaluate to an
`AliasSeq` (and thus could contain a combination of values, types, or symbols)
are PascalCased. Also, various names have been changed in order to improve
consistency. "static" and "template" have been removed from all of the names,
since they're not necessary, and neither is a convention that has ever been
followed consistently (even within std.meta). Presumably, in at least some of
the cases, it was done to prevent conflicts with symbols in std.algorithm, but
the module system already gives us a clean way to distinguish between symbols
with the same name but which are in different modules.

---
# Symbols

## Alias

* The Phobos v2 equivalent is [`std.meta.Alias`](../../std/meta.md#alias).
* Its functionality is unchanged.

## AliasSeq

* The Phobos v2 equivalent is [`std.meta.AliasSeq`](../../std/meta.md#aliasseq).
* Its functionality is unchanged.

## all

* The Phobos v2 equivalent is [`std.meta.allSatisfy`](../../std/meta.md#allsatisfy).
* It's been renamed to better match std.algorithm's `all`, and because having
  "Satisfy" in the name is redundant, since that's true of any template which evaluates
  to `true` or `false` based on a predicate.
* Its functionality is unchanged.

## And

* The Phobos v2 equivalent is [`std.meta.templateAnd`](../../std/meta.md#templateand).
* It's been renamed to better follow the official naming conventions.
* Its functionality is essentially the same. However, unlike
  [`templateAnd`](../../std/meta.md#templateand), `And` does not short-circuit its
  evaluation (something which std.meta in general is inconsistent about).

## any

* The Phobos v2 equivalent is [`std.meta.anySatisfy`](../../std/meta.md#anysatisfy).
* It's been renamed to better match std.algorithm's `any`, and because having
  "Satisfy" in the name is redundant, since that's true of any template which evaluates
  to `true` or `false` based on a predicate.
* Its functionality is unchanged.

## ApplyLeft

* The Phobos v2 equivalent is [`std.meta.ApplyLeft`](../../std/meta.md#applyleft).
* Its functionality is unchanged.

## ApplyRight

* The Phobos v2 equivalent is [`std.meta.ApplyRight`](../../std/meta.md#applyright).
* Its functionality is unchanged.

## Filter

* The Phobos v2 equivalent is [`std.meta.Filter`](../../std/meta.md#filter).
* Its functionality is unchanged.

## indexOf

* The Phobos v2 equivalent is [`std.meta.staticIndexOf`](../../std/meta.md#staticindexof).
* It's been renamed to better follow the official naming conventions.
* It now takes a template predicate to use for the comparison.

## Instantiate

* The Phobos v2 equivalent is [`std.meta.Instantiate`](../../std/meta.md#instantiate).
* Its functionality is unchanged.

## Map

* The Phobos v2 equivalent is
  [`std.meta.staticMap`](../../std/meta.md#staticmap).
* It's been renamed to better follow the official naming conventions.
* Its functionality is unchanged.

## Not

* The Phobos v2 equivalent is [`std.meta.templateNot`](../../std/meta.md#templatenot).
* It's been renamed to better follow the official naming conventions.
* Its functionality is essentially the same. However, unlike
  [`templateNot`](../../std/meta.md#templatenot), `Not` does not short-circuit its
  evaluation (something which std.meta in general is inconsistent about).

## Or

* The Phobos v2 equivalent is [`std.meta.templateOr`](../../std/meta.md#templateor).
* It's been renamed to better follow the official naming conventions.
* Its functionality is essentially the same. However, unlike
  [`templateOr`](../../std/meta.md#templateor), `Or` does not short-circuit its
  evaluation (something which std.meta in general is inconsistent about).

## Reverse

* The Phobos v2 equivalent is [`std.meta.Reverse`](../../std/meta.md#reverse).
* Its functionality is unchanged.

## Stride

* The Phobos v2 equivalent is [`std.meta.Stride`](../../std/meta.md#stride).
* Its functionality is unchanged.

## Unique

* The Phobos v2 equivalent is [`std.meta.NoDuplicates`](../../std/meta.md#noduplicates).
* It's been renamed to be more in line with std.algorithm's `uniq`, which will
  probably be renamed to `unique` due to Adam Wilson being against truncated
  names.
* It now takes a template predicate to use for the comparison.
