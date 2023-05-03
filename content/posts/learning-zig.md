---
title: "Learning Zig with Ziglings"
tags: [Zig]
categories: [TILs]
date: 2022-12-27
draft: false
---

[ziglings](https://github.com/ziyunli/ziglings)

# Built-in

The `@import()` function is built into Zig.
It returns a value which represents the imported code.

```zig
const std = @import("std");
```

# Types

## Arrays

*Is it allocated on the heap?*

Initialization with `[n]type{ ... }`

```zig
var foo = [_]u32{ 42, 108, 5423 };
```

Concatenate with `++`

```zig
var foo = [_]u32{ 42, 108, 5423 } ++ [_]u32{ 42, 108, 5423 };
```

Repeat with `**`

```zig
var foo = [_]u32{ 42, 108, 5423 } ** 2;
```

## Strings

Strings use double quotes. Zig stores strings as arrays of bytes.

```zig
var foo = "Hello, world!";
```

Multi-line string literals are supported with leading `\\` at the beginning of each line.

```zig
const foo =
  \\Hello,
  \\world!
```

## Enums

```zig
const Fruit = enum{ apple, pear, orange };
```

## Structs

## Pointers



# Conditions

## If

* If only takes boolean condition
* If statements are valid expressions

## While

Continue expression is optional.

```zig
while (condition) : (continue expression) {
  // ...
}
```

`continue` and `break` are supported in while loops.

## For

`for` works like iterators on arrays and slices.

```zig
for (items) |item, index| {
  // ...
}
```

## Switch

`switch` works like `match` expression in other languages.
Switch statements must be "exhaustive".
They are also valid expressions.

```zig
switch (players) {
  1 => print("1 player", .{}),
  2 => print("2 players", .{}),
  else => print("{} players", .{ players })
}
```

`unreachable` is a special value that can be used to indicate that a switch statement is exhaustive.


# Functions

Zig functions are private by default but the `main()` function should be public.
A function is declared public with the `pub` statement.

## Defer

`defer` is a statement that executes a function when the current scope is exited.
`errdefer` is a statement that executes a function when the current scope is exited with an error.


# Errors

Errors are created in "error sets", which works like an enum.

```zig
const Error = error{ Foo, Bar };
```

Zig support "error unions", which could be either a regular value OR an error from a set.

```zig
var text: MyErrorSet!Text = ...;
```

`!void` will let Zig to infer the error type, which is useful for `main()`.

## Error handling

`catch` catches an error and replace it with a default value.

```zig
foo = canFail() catch "bar"
```

`catch` can also capture the error and perform additional actions.

```zig
foo = canFail() catch |err| {
  if (err == FishError.TunaMalfunction) {
    ...
  }
}
```

`try` is a shorthand for `catch` to return the error.

```zig
canFail() catch |err| return err;

// is equivalent to

try canFail();
```

`if` can be used to check if an error is present.

```zig
if (canFail()) |value| {
  // ...
} else |err| switch {
  FishError.TunaMalfunction => ...,
  else => ...,
}
```
