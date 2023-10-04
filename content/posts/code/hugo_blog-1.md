---
title: "Hugo 搭建博客"
date: 2023-09-19T00:22:05+08:00
draft: true
---

# Hugo 搭建博客

Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

Hugo中文文档：<https://www.gohugo.org/>

Hugo官网: <https://gohugo.io/>

## 实操

### 相关链接

- [Hugo](https://gohugo.io/)
- [Hugo 中文文档](https://gohugo.io/zh/)

### 安装 Hugo

Mac下直接使用 Homebrew 安装：

```sh
brew install hugo
```

### 生成站点

```bash
创建项目
hugo new site myblog

cd myblog
安装主题
git clone https://github.com/flysnow-org/maupassant-hugo themes/maupassant

hugo server -t maupassant --buildDrafts

hugo new post/blog.md

hugo server -t maupassant --buildDrafts

hugo --theme=maupassant --baseUrl="https://qiaopengjun5162.github.io/" --buildDrafts

cd public
echo "# qiaopengjun5162.github.io" >> README.md
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/qiaopengjun5162/qiaopengjun5162.github.io.git
git push -u origin main
```

### 创建文章

创建第一篇博客

```bash
hugo new posts/first_blog.md
```

在本地启动网站

```bash
hugo server
```

启动server时应用主题

```bash
hugo server --theme=LoveIt --watch

```

参数说明：

```
* --theme 用于选择主题，如果在配置文件中选择了主题，这里就不需要使用了
* --buildDrafts 用于是否显示草稿文章
* --watch 用于实时监控变化，方便调试
```

访问：<http://localhost:1313/>

- 在项目根目录下使用 hugo 命令，会生成 public 目录
- 该目录下是有关 markdown 编译完成的 html 静态页面
