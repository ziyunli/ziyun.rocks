+++
title = "Compile Redis With Zig"
slug = "compile-redis-with-zig"
date = "2023-08-11T16:13:44-07:00"
+++

There is a [HN thread](https://news.ycombinator.com/item?id=37081833) about C development setup that I tried to follow for my electronics project.
At the end, I am not sure if I like the Docker+DevContainer+CMake setup.
I used CMake ~10 years ago during my graduate program but wasn't a big fan, and seems like I am still not today.
Not to mention the mental overhead of learning Docker and DevContainer.

But in the HN thread, it mentions [Zig as the build system for C/C++ projects](https://news.ycombinator.com/item?id=37082231).
I have toyed with Zig late last year around Christmas time, which is more about getting a taste of the syntax.
So I thought, why not give it a try.

As suggested in the thread, I tried [Loris Cro's C/C++/Zig Series](https://zig.news/kristoff/series/3).
The first step is to [compile Redis with Zig](https://zig.news/kristoff/compile-a-c-c-project-with-zig-368j).
The command we need is `make CC="zig cc" CXX="zig c++"`, which is telling make to use Zig as the compiler.
In the article, it suggests "you will need to use Zig 0.8.1 and commit be6ce8a of Redis".
But I decided to try with Redis 7.2 and the latest Zig 0.11 instead.

The error I ran into is:

```sh
MAKE fpconv
cd fpconv && /Library/Developer/CommandLineTools/usr/bin/make
zig cc  -Wall -Os -g  -c  fpconv_dtoa.c
ar rcs libfpconv.a fpconv_dtoa.o
    CC adlist.o
error: LTO is not yet supported with the Mach-O object format. More details: https://github.com/ziglang/zig/issues/8680
```

Looks like Zig 0.11 doesn't support LTO yet, but with a minor change to comment out `-flto` in `Makefile`, I was able to compile successfully!
