+++
title = "Running Mojo With Python"
slug = "running-mojo-with-python"
date = "2023-10-17T16:46:01-07:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["lab-notes", "mojo", "python"]
+++

I ran into an issue when I tried to run [llama2.mojo](https://github.com/tairov/llama2.mojo) with `mojo llama2.mojo stories15M.bin -s 100 -n 256 -t 0.5 -i "Mojo is a language"`:

```
num parallel workers: 6  SIMD width: 16
Mojo/Python interoperability error: Unable to locate a suitable libpython, please set `MOJO_PYTHON_LIBRARY`
Please submit a bug report to https://github.com/modularml/mojo/issues and include the crash backtrace along with all the relevant source codes.
Stack dump:
0.	Program arguments: mojo llama2.mojo stories15M.bin -s 100 -n 256 -t 0.5 -i "Mojo is a language"
#0 0x0000555932ecfa57 (/home/ziyunli/.modular/pkg/packages.modular.com_mojo/bin/mojo+0x5b6a57)
#1 0x0000555932ecd62e (/home/ziyunli/.modular/pkg/packages.modular.com_mojo/bin/mojo+0x5b462e)
#2 0x0000555932ed012f (/home/ziyunli/.modular/pkg/packages.modular.com_mojo/bin/mojo+0x5b712f)
#3 0x00007f0ce4442520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
[1]    18863 segmentation fault (core dumped)  mojo llama2.mojo stories15M.bin -s 100 -n 256 -t 0.5 -i "Mojo is a language"
```

Then I found this thread https://github.com/modularml/mojo/issues/551, which says that compiled Mojo programs that import Python code would fail with this message because Mojo currently does not embed the Python version into Mojo binaries. The thread has several suggestions to find the right `MOJO_PYTHON_LIBRARY` value and I ended with this:

```sh
find $(python3 -c 'import sysconfig; print(sysconfig.get_config_var("LIBDIR"))') -type f -name '*libpython*' | head -n 1
```

Drop it to `.zshrc` and I was able to run the program:

```
num parallel workers: 6  SIMD width: 16
checkpoint size:  60816028 [ 57 MB ] | n layers: 6 | vocab size: 32000
Mojo is a language that people like to talk. Hephones are very different from other people. He has a big book with many pictures and words. He likes to look at the pictures and learn new things.
One day, Mojo was playing with his friends in the park. They were running and laughing and having fun. Mojo told them about his book and his friends. They listened and looked at the pictures. Then, they saw a picture of a big, scary monster. They were very scared and ran away.
Mojo was sad that his book was gone. He told his friends about the monster and they all felt very sad. Mojo's friends tried to make him feel better, but nothing worked. Mojo never learned his language again.
achieved tok/s:  455.0561797752809
```
