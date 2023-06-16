---
title: "macOS notarization"
tags: ["til", "macos"]
date: 2022-08-26
draft: false
---


Sometimes you download a precompiled application that is not notarized or signed, and the first time you run the system would show a warning about the impossibility to check for malicious software. This is since macOS 10.15 “Catalina” when Apple introduced new notarization requirements [^1].

To fix, you can launch the app with right click (or CTRL click) on the app icon and choose the open action [^2].

Alternatively, run the following command from the terminal:

```shell
xattr -r -d com.apple.quarantine "FULL PATH OF app"
```

Do make sure the application is safe to run!

[^1]: https://firefox-source-docs.mozilla.org/testing/geckodriver/Notarization.html "macOS notarization"
[^2]: https://github.com/sbarex/QLMarkdown
