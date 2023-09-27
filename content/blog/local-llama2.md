+++
title = "Setting up LLaMa 2 locally with LLM CLI"
date = "2023-09-27T11:17:39-07:00"

tags = ["lab-notes","large-language-model"]
+++

I am trying to set up a local LLM to play with some ideas, and I started with [Simon Willison's post](https://simonwillison.net/2023/Aug/1/llama-2-mac/) that is based on his [LLM CLI tool](https://github.com/simonw/llm).

I am planning to eventually poke around the LLM CLI code, so instead of using Homebrew I used `pipx` to install to roughly make sense I have a working Python runtime. I also installed `symbex` and `datasette` so that I can play around with Simon's other tutorials.

```sh
pipx install llm
pipx upgrade symbex
pipx install datasette
```

Then I just followed the tutorial, it all worked until I tried to run the `llm` command and I got an empty error message:

```
❯ llm -m l2c 'Tell me a joke about a llama'
Error:
```

After some Kagi'ing, I found https://github.com/simonw/llm/issues/189 and learned that we can use `-o verbose true` to see the actual error.
And here we had:

```
❯ llm prompt -m l2c 'Tell me a joke about a llama' -o verbose true
gguf_init_from_file: invalid magic number 67676a74
error loading model: llama_model_loader: failed to load model from /Users/ziyunli/Library/Application Support/io.datasette.llm/llama-cpp/models/llama-2-7b-chat.ggmlv3.q8_0.bin

llama_load_model_from_file: failed to load model
Error:
```

This led me to another thread https://huggingface.co/TheBloke/Llama-2-13B-chat-GGML/discussions/14. This is due to the fact that llama.cpp has switched from GGML to GGUF. In the thread, it suggests to run a conversion script, but I decided to just download the GGUF model.

```sh
llm llama-cpp download-model \
  https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF/resolve/main/llama-2-7b-chat.Q8_0.gguf \
  --alias llama2-chat --alias l2c --llama2-chat
```

A bit of side note, but if you have already downloaded the GGML, then you might want to clean up the LLM config file to remove the alias for the old GGML model. You can find the config file by following https://llm.datasette.io/en/stable/setup.html#setting-a-custom-directory-location.

Now, it's working...well kind of:

```
❯ llm -m l2c 'Tell me a joke about a llama' --system 'You are Jerry Seinfeld'

 (In my best Jerry Seinfeld voice) Oh, boy...llamas. You know, I've been thinking, have you ever noticed how llamas are like the ultimate pretenders? They're always standing there, looking all regal and noble, but deep down they're just a bunch of woolly impostors. (chuckles) I mean, have you seen their ears? They're like little floppy antennae, trying to listen in on all the juicy gossip. (giggles) And don't even get me started on their spittingggml_metal_free: deallocating

❯ llm -c 'Now be George'
 Oh boy, oh boy! *excitedly* Llamas?! (clears throat) Well, you know what they say, "A llama in the hand is worth two in the bush!" *winks* But seriously, have you ever seen those fluffy creatures try to cross a busy street? They're like little traffic cones, trying to figure out which way the cars are gonna go! (chuckles) And don't even get me started on their love of soccer. I mean, they're always kicking the ball around, but they never scoreggml_metal_free: deallocating
```

For some reason, I always get this `ggml_metal_free: deallocating` string appended at the end of the response.
~~Is this some issue with the GGUF modell?~~
This seems to be an issue where C++ logging output is leaked to the STDOUT, https://github.com/nomic-ai/gpt4all/issues/1159 seems relevant.
If you look at the actual response in the logs via `datasette "$(llm logs path)"`, you will see the response is actually correct.
