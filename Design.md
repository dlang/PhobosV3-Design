# Phobos Version 3

## Rationale

Phobos v2 is well over a decade old. As time goes by, best practices have evolved, mistakes have become apparent, needs have changed, and better ideas have appeared.

Phobos must also evolve.

## History

* Phobos v1 was for D1
* Phobos v2 was for D2

## Ground Rules

### Phobos v2

#### Must continue to be supported

When Phobos v2 was developed, Phobos v1 was obsoleted. This broke a lot of existing code, and was a wrenching change. Old code needed rewrites to use Phobos v2. Some of the discarded v1 modules were restored in the unDead library for the convenience of older code, but the damage had been done.

Therefore, v2 will continue to be supported. A user must be able to mix and match v3 code with v2 code.

#### Will receive bug fixes and enhancements to adapt to new operating systems and new D Compiler Editions

#### Will not focus on new functionality

### Phobos v3

#### Existing Phobos V2 roots remain reserved indefinitely

The existing roots of `core` for the Runtime, `etc` for C Interfaces, and `std` for the V2 Standard Library are to remain reserved for the foreseeable future. Both V2 and V3 will share usage of the `core` and `etc` roots. The `std` root is to be maintained for compatibility but no new features will be added.

#### Hybrid single-root/multi-trunk package design

Phobos V3 will use a single package root `lib.` with multiple 'trunk' packages. This allows us to keep the old `std` package root while splitting up the new library into smaller, more manageable, components. Splitting the library into multiple packages provides the following benefits:

1. Only pay for what you use. Different roots can be compiled as separate static/shared libraries and linked as needed, reducing the overall weight of executables. This offers some relief to the long standing community request to break Phobos into individual packages. While we do not agree with atomizing Phobos to the extreme degree of one package per module, multiple package roots allow us to achieve some of that goal in a logical manner.
2. Multiple trunks allow for layering components. The core roots form a foundation upon which to build higher-level packages.
3. Multiple trunks allow for an expanded feature set without adding to package depth.
4. Not all trunks need to be implemented for a given platform to be considered "supported". If only the core roots are required for a platform to be considered supported it becomes significantly less complicated to port D to new platforms.
5. Rules can be applied differently to core vs. non-core roots/trunks. For example, a possible rule could be that core roots/trunks may not throw exceptions under any circumstance, but non-core roots/trunks can. This is not intended to imply that throwing exceptions is encouraged, merely made available to allow more implementation flexibility for more complex constructs.
6. The old `std` root can continue to be maintained and built independently of the new Phobos root.
7. Multiple trunk packages allows Phobos to expand normally without running into the 64k DLL symbol limit on Windows.

Currently the core roots/trunks for Phobos V3 are `core`, `etc`, and `phobos.sys`. As a rule, the `phobos.sys` trunk is not allowed to import from non-core trunks.

Proposed Package Structure (Existing Modules Only):
```
core.*
etc.*
std.*
phobos.sys
  | algorithm
    | comparison
    | iteration
    | mutation
    | searching
    | setops
    | sorting
  | array
  | bigint
  | bitmanip
  | checkedint
  | compiler
  | complex
  | conv
  | datetime
    | date
    | interval
    | stopwatch
    | systime
    | timezone
  | demangle
  | exception
  | functional
  | getopt
  | int128
  | io
    | console (stdio)
    | file
    | mmfile
    | path
  | math
    | algebraic
    | constants
    | exponential
    | hardware
    | operations
    | remainder
    | rounding
    | traits
    | trigonometry
    | special
  | meta
  | numeric
  | outbuffer
  | process
  | random
  | range
  | signals
  | stdint
  | string
  | sumtype
  | system
  | traits
  | typecons
  | uuid
  | variant
phobos.data
  | base64
  | csv
  | json
  | zip
phobos.text
  | ascii
  | encoding
  | uni
  | utf
phobos.net
  | socket
  | uri
```

**Open Questions**

1. `std.container`: Does it belong in `sys` or it's own root? I suspect this answer hinges on what rules are applied to it.
2. `std.concurrency`: Not all platforms support Concurrency mechanics. Should `std.concurrency` be required in the core roots or moved to a non-core root?
3. `std.parallelism`: Not all platforms support Task Parallelism mechanics. Should `std.parallelism` be required in the core roots or moved to a non-core root?
4. `std.digest`: Cryptography routines have unique requirements that would be best served by providing them in their own root. But in all cases providing bespoke implementations of cryptography routines is never recommended. Third-party trusted implementations should be used instead. This could be from OpenSSL/LibreSSL (POSIX), SChannel/BCrypt (Windows), or CryptoKit (MacOS/IOS). It is acceptable to provide implementations of Non-Cryptographic routines such as CRC and Murmur. This would entail removing all the digests except CRC and Murmur from this package and building a separate cryptography package.
5. Which, if any, modules should be removed? This could be either a permanent removal or pending replacement with ground-up rewrites.

## Overarching Technical Goals

### Strings

#### No Autodecoding

Autodecoding turned out to be a mistake because it is pervasive and impractical to disable. The user will need to specifically ask for decoding using a filter such as `utf.byDchar`.

#### No Support For wchar And dchar

Since v2 was designed, the programming world has more or less standardized on UTF-8. The internals of algorithms, ranges, and functions will only work with UTF-8. Support for `wchar` and `dchar` will come in the form of algorithms `utf.byChar`, `utf.byWchar` and `utf.byDchar`.

#### Invalid Unicode

v2 throws an Exception when encountering invalid Unicode. Throwing an Exception entails using the GC, meaning string code cannot be `@nogc` nor `nothrow`.
Besides, common processing of strings means being tolerant of invalid Unicode rather than failing. For example, invalid Unicode is commonplace in web pages, and throwing an Exception when rendering such pages is unacceptable.

Removal of autodecoding will in itself address most of the problem. When decoding code units into code points is needed, APIs should allow callers to specify the desired behavior, such as returning an "error" result, or replacing invalid sequences with the Unicode substitution character.
Applications which need to handle untrusted data should be encouraged to use functions such as `std.utf.validate` (which return a `string` from `ubyte[]` only when it is valid UTF-8), or by-code-point decoding which reports errors for individual decoding operations.

### Minimize Memory Allocation

Enormous troubles and inefficiencies stem from general purpose library code allocating memory as the library sees fit. This makes the library code far less usable. Memory allocation strategies should be decided upon by the user of the library, not the library.

The easiest way to achieve this is to design the library to not use memory allocation at all. For example, std.path assembles paths from parts without doing allocations - it returns Voldemort ranges that the user can then use to emit the result into a buffer of the user's choosing.

Library routines may allocate memory internally, but not in a way that affects the user.

### Minimize Exceptions

Exceptions are inefficient and use the GC. Of course, they cannot be used in `nothrow` code. Examine each use of an Exception to see if it can be designed out of existence, like the Replacement Character method above. Design the return value such that an error is not necessary - for example, a string search function can return an empty string if not found rather than throw an Exception.
Investigate the use of Option/Sum types for error returns.

### Source Only

Currently, Phobos is distributed as a separately compiled library with "header" files that contain only the necessary Template implementations. The net result is that Phobos is primarily a source-only library in practice. By formalizing the library as a source only library, it becomes inured against variations in compiler flags. Whether it's a static or dynamic library becomes irrelevant, and there won't be impedance mismatches. DRuntime will remain a separately compiled library.

### Naming and Style Guidelines

Names should follow the existing naming guidelines here: [D Style Guide](https://dlang.org/dstyle.html)

When selecting a name for a type or method, a quick survey of how other popular languages name the equivalent type/method should be performed. For example, in .NET and Java, the `currTime()` method would be named `now()`. Using the same names as popular languages reduces the friction experienced by the engineer when migrating to D. Be prepared to provide examples from your survey in the Pull Request. In cases where there is no clear agreement or two examples are equally represented and alias *may* be appropriate for the purposes of moving past the block. 

Prefer whole words over abbreviations and dropped letters. For example, prefer `writeLine` over `writeln`. Choose the shortest name that accurately describes the feature. Abbreviations are acceptable where the abbreviation is in common usage and/or would result in a cumbersome name.

Phobos will use a 100 soft and 120 hard character column limit. This will be enforced via `dfmt` and `.editorconfig` files provided with the distribution.

### DUB Only Builds

The current Phobos build process is convoluted and relies on old or niche tools. This makes the process of building Phobos tedious and discourages active community participation. DUB will be the only tool needed to build and test Phobos V3. Building and testing Phobos V3 should be as simple as issuing a `dub test` command. The complete CI process for Phobos3 will be a single build script that the developer can comprehend and execute easily.

### Versioning and Release Schedule

Phobos3 versions will be versioned and distributed on the same schedule as the corresponding Compiler Edition. This allows Phobos to adopt the latest features from the in-development edition during the development of that edition. However, while Phobos will follow the same release schedule as the compiler editions, Phobos itself will not use the 'edition' terminology and will instead retain the use of the term 'version'.

Phobos will use a slightly modified SemVer. The major version will increment with each Phobos release that is tied to a compiler edition, the minor version will increment with the monthly compiler releases, and the patch version will be incremented on any out-of-band bugfix releases that occur.

## Specific Issues

### std.stdio

This module merges file I/O with formatting. Those should be split apart. File I/O should be done with ranges, and the formatting should work with any ranges.

stdin, stdout, and stderr match the stdin, stdout and stderr of core.stdc.stdio.
This causes terrible confusion. Should be stdIn, stdOut, and stdErr.

#### std.stdio.File

Does way, way too much. It's incomprehensible. Should be redesigned using building blocks.

### isXXXX templates

Are generally very hard to figure out what they do, such as `isSomeChar`. What the heck does that mean? The string ones are even worse.

## Guidelines for Reviewers

### Silence is Approval
One of the complaints the community has made about D in general and Phobos in particular is that many PRs go un-reviewed and are left to rot. Therefore, in Phobos 3, if you are listed as a reviewer on a PR and you do not respond to a within 24 hours to a pull request, your silence will be considered an approval. If the required reviewer is unavailable for any reason (ex: vacation, emergency, etc.), another reviewer can bypass them. Once they return they can provide comments and submit PR's to address any feedback on the PR's assigned to them that they missed.

### Disagreement Without Providing a Fix or Alternative is Approval
Another major community complaint is that PR's are routinely abandoned because somebody disagrees with how the PR does something, or believes that it will cause a problem, causing the PR to stall out. While the reviewer may be correct, that cannot be sufficient reason to block a PR. The disagreeing reviewer must either provide a PR/patch to the primary PR or provide an alternative implementation in a new PR that addresses their concern. The disagreeing reviewer must ensure that their PR references the original PR. If no alternatives are provided, the PR will be merged and the disagreeing reviewer is welcome to base their future alternative implementation on the merged work.
