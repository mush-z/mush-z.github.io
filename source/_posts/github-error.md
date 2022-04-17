---
title: github-error
date: 2022-04-17 14:01:51
tags: github+hexo error
category: Linux
---

## 1.error

```
fatal: 不是 git 仓库（或者任何父目录）：.git
```

解决:

```shell
vim _config.yml

highlight:
  enable: flash
```



## 2.error

```shell
ERROR Deployer not found: git
```

解决:

```shell
npm install hexo-deployer-git --save
```
