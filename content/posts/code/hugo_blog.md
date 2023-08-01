---
title: "Hugo 搭建博客"
date: 2023-07-29T12:09:39+08:00
draft: true
---

# Hugo 搭建博客



```bash
~/blog via 🅒 base
➜ hugo new site qiaoblog -f yml
Congratulations! Your new Hugo site is created in /Users/qiaopengjun/blog/qiaoblog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.

~/blog via 🅒 base
➜ cd qiaoblog
```



问题：connect to github.com port 443

解决：

```bash
git config --global url."https://ghproxy.com/https://github.com".insteadOf "https://github.com"
```



```bash
~/blog/qiaoblog via 🅒 base
➜ hugo
Start building sites …
hugo v0.110.0+extended darwin/arm64 BuildDate=unknown

                   | EN
-------------------+------
  Pages            |  23
  Paginator pages  |   0
  Non-page files   |   0
  Static files     | 368
  Processed images |   0
  Aliases          |   5
  Sitemaps         |   1
  Cleaned          |   0

Total in 110 ms

~/blog/qiaoblog via 🅒 base
➜ ls
archetypes config.yml data       layouts    resources  themes
assets     content    i18n       public     static

~/blog/qiaoblog via 🅒 base
➜ cd public

blog/qiaoblog/public via 🅒 base
➜ echo "# qiao.github.io" >> README.md

blog/qiaoblog/public via 🅒 base
➜ git init
提示：使用 'master' 作为初始分支的名称。这个默认分支名称可能会更改。要在新仓库中
提示：配置使用初始分支名，并消除这条警告，请执行：
提示：
提示：	git config --global init.defaultBranch <名称>
提示：
提示：除了 'master' 之外，通常选定的名字有 'main'、'trunk' 和 'development'。
提示：可以通过以下命令重命名刚创建的分支：
提示：
提示：	git branch -m <name>
已初始化空的 Git 仓库于 /Users/qiaopengjun/blog/qiaoblog/public/.git/

public on  master [?] via 🅒 base
➜ git add .

public on  master [+] via 🅒 base
➜ git commit -m "first commit"
[master（根提交） 983de54] first commit
 401 files changed, 136779 insertions(+)
 create mode 100644 .DS_Store
 create mode 100644 404.html
 create mode 100644 README.md
 create mode 100644 about/index.html
 create mode 100644 archives/index.html
 create mode 100644 assets/css/stylesheet.css
 create mode 100644 assets/js/highlight.js
 create mode 100644 assets/js/search.js
 create mode 100644 categories/index.html
 create mode 100644 categories/index.xml
 create mode 100644 img/.DS_Store
 create mode 100644 img/Q.gif
 create mode 100644 index.html
 create mode 100644 index.json
 create mode 100644 index.xml
 create mode 100644 js/.DS_Store
 create mode 100644 js/pdf-js/.DS_Store
 create mode 100644 js/pdf-js/LICENSE
 create mode 100644 js/pdf-js/build/pdf.js
 create mode 100644 js/pdf-js/build/pdf.js.map
 create mode 100644 js/pdf-js/build/pdf.sandbox.js
 create mode 100644 js/pdf-js/build/pdf.sandbox.js.map
 create mode 100644 js/pdf-js/build/pdf.worker.js
 create mode 100644 js/pdf-js/build/pdf.worker.js.map
 create mode 100644 js/pdf-js/web/_url_.pdf
 create mode 100644 js/pdf-js/web/cmaps/78-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78ms-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/78ms-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/83pv-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90ms-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90ms-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90msp-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90msp-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90pv-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/90pv-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Add-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Add-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Add-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Add-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-0.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-1.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-3.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-4.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-5.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-6.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-CNS1-UCS2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-0.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-1.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-3.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-4.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-5.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-GB1-UCS2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-0.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-1.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-3.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-4.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-5.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-6.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Japan1-UCS2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Korea1-0.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Korea1-1.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Korea1-2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Adobe-Korea1-UCS2.bcmap
 create mode 100644 js/pdf-js/web/cmaps/B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/B5pc-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/B5pc-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS1-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS1-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS2-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/CNS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETHK-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETHK-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETen-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETen-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETenms-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/ETenms-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Ext-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Ext-RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Ext-RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Ext-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GB-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GB-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GB-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GB-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBK-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBK-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBK2K-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBK2K-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBKp-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBKp-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBT-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBT-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBT-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBT-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBTpc-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBTpc-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBpc-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/GBpc-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKdla-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKdla-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKdlb-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKdlb-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKgccs-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKgccs-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKm314-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKm314-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKm471-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKm471-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKscs-B5-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/HKscs-B5-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Hankaku.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Hiragana.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-Johab-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-Johab-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCms-UHC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCms-UHC-HW-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCms-UHC-HW-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCms-UHC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCpc-EUC-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/KSCpc-EUC-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Katakana.bcmap
 create mode 100644 js/pdf-js/web/cmaps/LICENSE
 create mode 100644 js/pdf-js/web/cmaps/NWP-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/NWP-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/RKSJ-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/RKSJ-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/Roman.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UCS2-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UCS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF16-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF16-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF8-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniCNS-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UCS2-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UCS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF16-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF16-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF8-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniGB-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UCS2-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UCS2-HW-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UCS2-HW-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UCS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF16-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF16-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF8-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF16-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF16-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF8-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJIS2004-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISPro-UCS2-HW-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISPro-UCS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISPro-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISX0213-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISX0213-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISX02132004-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniJISX02132004-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UCS2-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UCS2-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF16-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF16-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF32-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF32-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF8-H.bcmap
 create mode 100644 js/pdf-js/web/cmaps/UniKS-UTF8-V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/V.bcmap
 create mode 100644 js/pdf-js/web/cmaps/WP-Symbol.bcmap
 create mode 100644 js/pdf-js/web/debugger.js
 create mode 100644 js/pdf-js/web/images/annotation-check.svg
 create mode 100644 js/pdf-js/web/images/annotation-comment.svg
 create mode 100644 js/pdf-js/web/images/annotation-help.svg
 create mode 100644 js/pdf-js/web/images/annotation-insert.svg
 create mode 100644 js/pdf-js/web/images/annotation-key.svg
 create mode 100644 js/pdf-js/web/images/annotation-newparagraph.svg
 create mode 100644 js/pdf-js/web/images/annotation-noicon.svg
 create mode 100644 js/pdf-js/web/images/annotation-note.svg
 create mode 100644 js/pdf-js/web/images/annotation-paragraph.svg
 create mode 100644 js/pdf-js/web/images/findbarButton-next.svg
 create mode 100644 js/pdf-js/web/images/findbarButton-previous.svg
 create mode 100644 js/pdf-js/web/images/grab.cur
 create mode 100644 js/pdf-js/web/images/grabbing.cur
 create mode 100644 js/pdf-js/web/images/loading-dark.svg
 create mode 100644 js/pdf-js/web/images/loading-icon.gif
 create mode 100644 js/pdf-js/web/images/loading.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-documentProperties.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-firstPage.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-handTool.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-lastPage.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-rotateCcw.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-rotateCw.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-scrollHorizontal.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-scrollPage.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-scrollVertical.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-scrollWrapped.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-selectTool.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-spreadEven.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-spreadNone.svg
 create mode 100644 js/pdf-js/web/images/secondaryToolbarButton-spreadOdd.svg
 create mode 100644 js/pdf-js/web/images/shadow.png
 create mode 100644 js/pdf-js/web/images/toolbarButton-bookmark.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-currentOutlineItem.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-download.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-menuArrow.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-openFile.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-pageDown.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-pageUp.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-presentationMode.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-print.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-search.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-secondaryToolbarToggle.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-sidebarToggle.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-viewAttachments.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-viewLayers.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-viewOutline.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-viewThumbnail.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-zoomIn.svg
 create mode 100644 js/pdf-js/web/images/toolbarButton-zoomOut.svg
 create mode 100644 js/pdf-js/web/images/treeitem-collapsed.svg
 create mode 100644 js/pdf-js/web/images/treeitem-expanded.svg
 create mode 100644 js/pdf-js/web/locale/ach/viewer.properties
 create mode 100644 js/pdf-js/web/locale/af/viewer.properties
 create mode 100644 js/pdf-js/web/locale/an/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ar/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ast/viewer.properties
 create mode 100644 js/pdf-js/web/locale/az/viewer.properties
 create mode 100644 js/pdf-js/web/locale/be/viewer.properties
 create mode 100644 js/pdf-js/web/locale/bg/viewer.properties
 create mode 100644 js/pdf-js/web/locale/bn/viewer.properties
 create mode 100644 js/pdf-js/web/locale/bo/viewer.properties
 create mode 100644 js/pdf-js/web/locale/br/viewer.properties
 create mode 100644 js/pdf-js/web/locale/brx/viewer.properties
 create mode 100644 js/pdf-js/web/locale/bs/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ca/viewer.properties
 create mode 100644 js/pdf-js/web/locale/cak/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ckb/viewer.properties
 create mode 100644 js/pdf-js/web/locale/cs/viewer.properties
 create mode 100644 js/pdf-js/web/locale/cy/viewer.properties
 create mode 100644 js/pdf-js/web/locale/da/viewer.properties
 create mode 100644 js/pdf-js/web/locale/de/viewer.properties
 create mode 100644 js/pdf-js/web/locale/dsb/viewer.properties
 create mode 100644 js/pdf-js/web/locale/el/viewer.properties
 create mode 100644 js/pdf-js/web/locale/en-CA/viewer.properties
 create mode 100644 js/pdf-js/web/locale/en-GB/viewer.properties
 create mode 100644 js/pdf-js/web/locale/en-US/viewer.properties
 create mode 100644 js/pdf-js/web/locale/eo/viewer.properties
 create mode 100644 js/pdf-js/web/locale/es-AR/viewer.properties
 create mode 100644 js/pdf-js/web/locale/es-CL/viewer.properties
 create mode 100644 js/pdf-js/web/locale/es-ES/viewer.properties
 create mode 100644 js/pdf-js/web/locale/es-MX/viewer.properties
 create mode 100644 js/pdf-js/web/locale/et/viewer.properties
 create mode 100644 js/pdf-js/web/locale/eu/viewer.properties
 create mode 100644 js/pdf-js/web/locale/fa/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ff/viewer.properties
 create mode 100644 js/pdf-js/web/locale/fi/viewer.properties
 create mode 100644 js/pdf-js/web/locale/fr/viewer.properties
 create mode 100644 js/pdf-js/web/locale/fy-NL/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ga-IE/viewer.properties
 create mode 100644 js/pdf-js/web/locale/gd/viewer.properties
 create mode 100644 js/pdf-js/web/locale/gl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/gn/viewer.properties
 create mode 100644 js/pdf-js/web/locale/gu-IN/viewer.properties
 create mode 100644 js/pdf-js/web/locale/he/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hi-IN/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hr/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hsb/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hu/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hy-AM/viewer.properties
 create mode 100644 js/pdf-js/web/locale/hye/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ia/viewer.properties
 create mode 100644 js/pdf-js/web/locale/id/viewer.properties
 create mode 100644 js/pdf-js/web/locale/is/viewer.properties
 create mode 100644 js/pdf-js/web/locale/it/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ja/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ka/viewer.properties
 create mode 100644 js/pdf-js/web/locale/kab/viewer.properties
 create mode 100644 js/pdf-js/web/locale/kk/viewer.properties
 create mode 100644 js/pdf-js/web/locale/km/viewer.properties
 create mode 100644 js/pdf-js/web/locale/kn/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ko/viewer.properties
 create mode 100644 js/pdf-js/web/locale/lij/viewer.properties
 create mode 100644 js/pdf-js/web/locale/lo/viewer.properties
 create mode 100644 js/pdf-js/web/locale/locale.properties
 create mode 100644 js/pdf-js/web/locale/lt/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ltg/viewer.properties
 create mode 100644 js/pdf-js/web/locale/lv/viewer.properties
 create mode 100644 js/pdf-js/web/locale/meh/viewer.properties
 create mode 100644 js/pdf-js/web/locale/mk/viewer.properties
 create mode 100644 js/pdf-js/web/locale/mr/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ms/viewer.properties
 create mode 100644 js/pdf-js/web/locale/my/viewer.properties
 create mode 100644 js/pdf-js/web/locale/nb-NO/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ne-NP/viewer.properties
 create mode 100644 js/pdf-js/web/locale/nl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/nn-NO/viewer.properties
 create mode 100644 js/pdf-js/web/locale/oc/viewer.properties
 create mode 100644 js/pdf-js/web/locale/pa-IN/viewer.properties
 create mode 100644 js/pdf-js/web/locale/pl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/pt-BR/viewer.properties
 create mode 100644 js/pdf-js/web/locale/pt-PT/viewer.properties
 create mode 100644 js/pdf-js/web/locale/rm/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ro/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ru/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sat/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sc/viewer.properties
 create mode 100644 js/pdf-js/web/locale/scn/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sco/viewer.properties
 create mode 100644 js/pdf-js/web/locale/si/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sk/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/son/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sq/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sr/viewer.properties
 create mode 100644 js/pdf-js/web/locale/sv-SE/viewer.properties
 create mode 100644 js/pdf-js/web/locale/szl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ta/viewer.properties
 create mode 100644 js/pdf-js/web/locale/te/viewer.properties
 create mode 100644 js/pdf-js/web/locale/tg/viewer.properties
 create mode 100644 js/pdf-js/web/locale/th/viewer.properties
 create mode 100644 js/pdf-js/web/locale/tl/viewer.properties
 create mode 100644 js/pdf-js/web/locale/tr/viewer.properties
 create mode 100644 js/pdf-js/web/locale/trs/viewer.properties
 create mode 100644 js/pdf-js/web/locale/uk/viewer.properties
 create mode 100644 js/pdf-js/web/locale/ur/viewer.properties
 create mode 100644 js/pdf-js/web/locale/uz/viewer.properties
 create mode 100644 js/pdf-js/web/locale/vi/viewer.properties
 create mode 100644 js/pdf-js/web/locale/wo/viewer.properties
 create mode 100644 js/pdf-js/web/locale/xh/viewer.properties
 create mode 100644 js/pdf-js/web/locale/zh-CN/viewer.properties
 create mode 100644 js/pdf-js/web/locale/zh-TW/viewer.properties
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitDingbats.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitFixed.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitFixedBold.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitFixedBoldItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitFixedItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSans.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSansBold.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSansBoldItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSansItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSerif.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSerifBold.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSerifBoldItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSerifItalic.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/FoxitSymbol.pfb
 create mode 100644 js/pdf-js/web/standard_fonts/LICENSE_FOXIT
 create mode 100644 js/pdf-js/web/standard_fonts/LICENSE_LIBERATION
 create mode 100644 js/pdf-js/web/standard_fonts/LiberationSans-Bold.ttf
 create mode 100644 js/pdf-js/web/standard_fonts/LiberationSans-BoldItalic.ttf
 create mode 100644 js/pdf-js/web/standard_fonts/LiberationSans-Italic.ttf
 create mode 100644 js/pdf-js/web/standard_fonts/LiberationSans-Regular.ttf
 create mode 100644 js/pdf-js/web/viewer.css
 create mode 100644 js/pdf-js/web/viewer.html
 create mode 100644 js/pdf-js/web/viewer.js
 create mode 100644 js/pdf-js/web/viewer.js.map
 create mode 100644 links/index.html
 create mode 100644 posts/book/index.html
 create mode 100644 posts/book/index.xml
 create mode 100644 posts/book/page/1/index.html
 create mode 100644 posts/code/index.html
 create mode 100644 posts/code/index.xml
 create mode 100644 posts/code/page/1/index.html
 create mode 100644 posts/index.html
 create mode 100644 posts/index.xml
 create mode 100644 posts/life/index.html
 create mode 100644 posts/life/index.xml
 create mode 100644 posts/life/page/1/index.html
 create mode 100644 posts/music/index.html
 create mode 100644 posts/music/index.xml
 create mode 100644 posts/music/page/1/index.html
 create mode 100644 posts/page/1/index.html
 create mode 100644 robots.txt
 create mode 100644 search/index.html
 create mode 100644 sitemap.xml
 create mode 100644 tags/index.html
 create mode 100644 tags/index.xml

public on  master via 🅒 base
➜ git remote add origin git@github.com:qiaopengjun5162/qiao.github.io.git

public on  master via 🅒 base
➜ git branch -M main

public on  main via 🅒 base
➜ git push -u origin main
枚举对象中: 545, 完成.
对象计数中: 100% (545/545), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (420/420), 完成.
写入对象中: 100% (545/545), 4.95 MiB | 1.64 MiB/s, 完成.
总共 545（差异 206），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (206/206), done.
To github.com:qiaopengjun5162/qiao.github.io.git
 * [new branch]      main -> main
分支 'main' 设置为跟踪 'origin/main'。

public on  main via 🅒 base took 8.0s
➜
```

