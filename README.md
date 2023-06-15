# [Ziyun Rocks](https://blog.ziyun.rocks)

[![Netlify Status](https://api.netlify.com/api/v1/badges/ab9865d6-cd79-4ace-881a-3aac9d538a2d/deploy-status)](https://app.netlify.com/sites/cute-kataifi-dd101a/deploys)

## Switching to PaperMod

### Bootstrap

```sh
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
cd themes/PaperMod
git fetch --tags
git checkout tags/v7.0
```

### After clone

```sh
cd themes/hugo-bearblog
git submodule update --init --recursive
```

## Instructions

### Serving over local network

```sh
hugo server --bind 192.168.50.114 --baseURL http://192.168.50.114 --port 8080
```
