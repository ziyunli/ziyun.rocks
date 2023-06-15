---
title: "Open Ruby Gem in an Editor"
tags: ["til", "ruby"]
date: 2022-08-02
draft: false
---

[bundle open](https://bundler.io/man/bundle-open.1.html) opens a gem in an editor. This is useful when you need to look into the source code of a gem that your application depends on.

The documentation above says that

> For this to work the `EDITOR` or `BUNDLER_EDITOR` environment variable has to be set.

This means this is likely to open up a VIM-variant editor for you. However, if you set `VISUAL` in your environment, the command will use that instead. For example, to open in Visual Studio Code, you can push the following in your `rc` file.

```
export VISUAL='code'
```

[Here](https://unix.stackexchange.com/questions/4859/visual-vs-editor-what-s-the-difference) has more discussions between `VISUAL` and `EDITOR` environment variables.
