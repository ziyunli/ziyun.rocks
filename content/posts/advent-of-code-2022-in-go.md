---
title: "Learning Go with Advent of Code 2022"
subtitle: ""
tags: [Golang]
categories: [TILs]
date: 2022-12-22T18:54:24-08:00
featured: false
draft: true
projects: ["Advent of Code 2022"]
---

In this year of [Advent of Code](https://adventofcode.com/2022) (AoC), I decide to use Go a try. Here collects the TILs during the process.

<!--more-->

# Getting Started

I used Tim Hockin's [go-build-template](https://github.com/thockin/go-build-template) as the reference to start the project. There is another [Standard Go Project Layout](https://github.com/golang-standards/project-layout) that seems totally overkill for Advent of Code, but could be useful when building a real application. Below is how the project looks.

```bash
% exa -T                                                                                                                                                                                                                                                               (main)
.
├── cmd
│  └── cli
│     └── main.go
├── input
│  └── 2022
│     ├── day01.txt
│     └── day02.txt
├── internal
│  └── year2022
│     ├── day1.go
│     ├── day2.go
│     └── year2022.go
├── go.mod
├── go.sum
├── LICENSE
└── README.md
```

I don't expect anyone to import anything from this repo. Therefore I put the solutions inside `internal`, which has special handling for Go projects ([more details on `internal packages`](https://docs.google.com/document/d/1e8kOo3r51b2BWtTs_1uADIA5djfXhPT36s6eHVRIvaU/edit)).

I also build a small CLI tool to run the solutions. Building CLI application is straightforward with the [`flag` package](https://pkg.go.dev/flag). I decided to read the input from STDIN by default. This can be done by using `bufio.Scanner`[^1]. This helps to test the solutions with the smaller example inputs, as below. Note that on macOS, you need to type `Ctrl+D` to signify the end of the input[^2].

```shell
% go run ./cmd/cli -year 2022 -day 1 -part 1
Year: 2022, Day: 1, Part: 1
1000
2000
3000

4000

5000
6000

7000
8000
9000

10000
Top most calories: [24000]
```

It's also possible to pipe the input to STDIN, which is what I do when running the full input. For example:
```shell
cat input/2022/day02.txt | go run ./cmd/cli -year 2022 -day 2 -part 2
```

# Go Basics

[Go by Example](https://gobyexample.com/) is a very useful reference.

## Operators

In Go, `%` computes the "remainder" as opposed to the "modulus"[^5]. Therefore, unlike other languages like Python, `%` can return negative numbers in Go. Below is a branchless implementation of Python's `%` operator.

```go
func mod(a, b int) int {
    return (a % b + b) % b
}
```

## Slices

### Sorting

Coming from Python and Ruby, sorting in Go requires several more steps and feels slightly cumbersome. There is [sort.Ints](http://golang.org/pkg/sort/#Ints) to sorts a slice of `int`s, but only in increasing order. Generally, you need to implement the [sort.Interface](http://golang.org/pkg/sort/#Interface) interface if you want to sort something and [sort.Reverse](http://golang.org/pkg/sort/#Reverse) just returns a different implementation of that interface that redefines the `Less` method[^3]. At the end, the following does the reverse, but I am still wrapping my head around it.

```go
sort.Sort(sort.Reverse(sort.IntSlice(numbers)))
```

### Concatenation

This is much easier because `append` is a variadic function, therefore you can just do it with three-dots[^6]:

```go
append([]int{1,2}, []int{3,4}...)
```

## Strings

In AoC, the input comes from an input file, and it's common to manipulate each line for the tasks. The [`strings` package](https://pkg.go.dev/strings) has plenty of useful methods to handle strings.

Strings are constructed by *runes*, identified by single-quotes, e.g. `'a'`. To convert a slice of runes to a string, you can simply use `string`, e,g,

```go
s := string([]rune{ 'a', 'b', 'c' })
```

### Regexp

Package [`regexp`](https://pkg.go.dev/regexp) implements regular expression search. It's based on RE2 syntax[^7]. Example for matching with capturing groups:

```go
r, _ := regexp.Compile(`move ([\d]+) from ([\d]+) to ([\d]+)`)
matches := r.FindStringSubmatch(line)
log.Printf("move %v from %v to %v", matches[1], matches[2], matches[3])
```

## Enum

There is real enums in Go, and looks like people work around it by using constants and type definition. Below is an example:
```go
type Color int

const (
	Red Color = iota
	Orange
	Yellow
	Green
	Blue
	Indigo
	Violet
)
```

It also uses the *constant generator* `iota`. In a `const` declaration, the value of `iota` begins at zero and increments by one for each item in the sequence[^4]. You can also use `iota` in more complex expressions, e.g. building bit masks.

It seems like you can also use the [`stringer`](https://pkg.go.dev/golang.org/x/tools/cmd/stringer)  package to generate `String()` methods for your enum type. But I haven't figured out how to use it yet.

[^1]: https://stackoverflow.com/questions/20895552/how-can-i-read-from-standard-input-in-the-console "How can I read from standard input in the console?"
[^2]: https://unix.stackexchange.com/questions/16333/how-to-signal-the-end-of-stdin-input "How to signal the end of stdin input("
[^3]: https://stackoverflow.com/questions/18343208/how-do-i-reverse-sort-a-slice-of-integer-go "How do I reverse sort a slice of integer Go?"
[^4]: https://www.gopl.io/ "The Go Programming Language"
[^5]: https://github.com/golang/go/issues/448#issuecomment-66049769 "Modulus returns negative numbers"
[^6]: https://stackoverflow.com/questions/16248241/concatenate-two-slices-in-go "Concatenate two slices in Go"
[^7]: https://golang.org/s/re2syntax "RE2 Syntax"
