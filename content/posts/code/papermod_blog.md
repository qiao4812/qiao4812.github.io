---
title: "Papermod_blog"
date: 2023-06-29T16:20:12+08:00
draft: false
tags: ["Blog"]
categories: ["Blog"]
---



```bash
~ via 🅒 base
➜ hugo new site wowchemyblog
Congratulations! Your new Hugo site is created in /Users/qiaopengjun/wowchemyblog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.

~ via 🅒 base
➜ cd wowchemyblog

~/wowchemyblog via 🅒 base
➜ git init

提示：使用 'master' 作为初始分支的名称。这个默认分支名称可能会更改。要在新仓库中
提示：配置使用初始分支名，并消除这条警告，请执行：
提示：
提示： git config --global init.defaultBranch <名称>
提示：
提示：除了 'master' 之外，通常选定的名字有 'main'、'trunk' 和 'development'。
提示：可以通过以下命令重命名刚创建的分支：
提示：
提示： git branch -m <name>
已初始化空的 Git 仓库于 /Users/qiaopengjun/wowchemyblog/.git/

wowchemyblog on  master [?] via 🅒 base
➜ cd ..

~ via 🅒 base
➜ mv wowchemyblog qiaoblog

~ via 🅒 base
➜ cd qiaoblog

qiaoblog on  master [?] via 🅒 base
➜ ls
archetypes  config.toml data        public      themes
assets      content     layouts     static

qiaoblog on  master [?] via 🅒 base
➜ git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1

正克隆到 'themes/PaperMod'...
致命错误：无法访问 'https://github.com/adityatelange/hugo-PaperMod/'：HTTP/2 stream 1 was not closed cleanly before end of the underlying stream

qiaoblog on  master [?] via 🅒 base took 2m 0.3s
➜ git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1

正克隆到 'themes/PaperMod'...
致命错误：无法访问 'https://github.com/adityatelange/hugo-PaperMod/'：Recv failure: Operation timed out

qiaoblog on  master [?] via 🅒 base took 1m 56.9s
➜ git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1

正克隆到 'themes/PaperMod'...
remote: Enumerating objects: 130, done.
remote: Counting objects: 100% (130/130), done.
remote: Compressing objects: 100% (111/111), done.
remote: Total 130 (delta 29), reused 52 (delta 13), pack-reused 0
接收对象中: 100% (130/130), 255.04 KiB | 154.00 KiB/s, 完成.
处理 delta 中: 100% (29/29), 完成.

qiaoblog on  master [?] via 🅒 base took 3.4s
➜ ls
archetypes  config.toml data        public      themes
assets      content     layouts     static

qiaoblog on  master [?] via 🅒 base
```
