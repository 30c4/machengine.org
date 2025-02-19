---
title: "Style guide"
description: "The Mach code style guide; Fast doesn't mean reckless, we follow these rules to keep our code readable and high-quality."
draft: false
layout: "docs"
docs_type: "about"
rss_ignore: true
---

# Mach code style guide

## Use `mach.testing` where applicable

In the [main Mach repository](https://github.com/hexops/mach) we have the [`mach.testing`](https://github.com/hexops/mach/blob/main/src/testing.zig) module, which provides alternatives to `std.testing` equality methods with:

* Less [ambiguity about order of parameters](https://github.com/ziglang/zig/issues/4437)
* Approximate absolute float equality by default
* Handling of vector and matrix types

You should use `mach.testing` wherever it is accessible (without pulling in an additional dependency.)

## `usingnamespace` is banned

`usingnamespace` in general is banned from use in Mach code, because it allows one to 'mixin' namespaces and tends to produce increasingly worse code quality over time. See e.g. [ziglang/zig#9618](https://github.com/ziglang/zig/pull/9618), [hexops/mach#396](https://github.com/hexops/mach/issues/796)

We've successfully banished `usingnamespace` in general, and have yet to encounter a single instance where removing it did not improve code quality & lead to better / more legible layout of code.

```zig
// This is prohibited!
pub usingnamespace @import("foo.zig");

// This is allowed
pub const bar = @import("foo.zig").bar;
pub const baz = @import("foo.zig").baz;
// ...
```

If you find that writing all of the `pub const y = x.y` is too tedious, and you're doing it 50+ times or something - that is a golden signal that your code layout is poor and you should move `y` directly into whatever file you are writing `pub const y = x.y` in.

**Exception:** Sometimes you really, genuinely do require a way to express a comptime struct mixin. This is only applicable when you are trying to e.g. create structs at comptime, not when trying to structure code. These cases should be extraordinarily rare. For example, mach-core requires [one such usage](https://github.com/hexops/mach-core/blob/50930539c32630b8054ecd48b443c7e336780da4/src/platform/native/main.zig#L6-L7) because of magic `@import("root")` usages to configure e.g. logging in the Zig standard library.

## Test names should be snake_case and descriptive

Do this:

```
test "vec_dot" { ... }
test "vec_dot_zero_inputs" { ... }
```

Not this:

```
test "vec.dot" { ... }
test "vec.dot(0)" { ... }
```

## Tests should be small and contain a single test only

Do this:

```
test "vec_dot" {
    // test 1
}
test "vec_dot_zero_inputs" {
    // test 2
}
```

Not this:

```
test "vec.dot" {
    {
        // test 1
    }
    {
        // test 2
    }
}
```
