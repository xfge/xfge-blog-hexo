---
layout: post
title: "重新安装 macOS 的配置"
date: 2017-10-11
tags: [macos]
categories:
  - 日常
  - 备忘
---

## 清单

- iTerm2 + oh-my-zsh
- Java 及其配置
- Latex + Atom
- Aria2GUI
- GitHub Pages
- Maven 及其配置
- 其他


## 细节

### iTerm2 + oh-my-zsh

1. 安装 [Homebrew](https://brew.sh)。
2. 安装 iTerm2 和 Oh My Zsh 参考 [iTerm 2 && Oh My Zsh【DIY教程——亲身体验过程】 - 简书](http://www.jianshu.com/p/7de00c73a2bb)。
3. 语法高亮和自动提示参考 [Mac下终端配置（item2 + oh-my-zsh + solarized配色方案） - 希希里之海 - 博客园](http://www.cnblogs.com/weixuqin/p/7029177.html)。


### Java 及其配置

1. 直接在 `~/.bash_profile`（用户级） 或者 `~/.profile`（系统级） 文件中设置 `JAVA_HOME`。

   ```shell
   $ vim .bash_profile

   export JAVA_HOME=$(/usr/libexec/java_home)

   $ source .bash_profile

   $ echo $JAVA_HOME
   /Library/Java/JavaVirtualMachines/1.7.0.jdk/Contents/Home
   ```

2. 因为使用了 zsh 替代 bash，因此要在 `.zshrc` 中加入一行 `source ~/.bash_profile`。


### Latex + Atom

1. 下载并安装 [MacTex](https://www.tug.org/mactex/)。
2. 在 Atom 中安装插件
  - latex
  - language-latex
  - pdf-view
3. 通常无需特殊配置即可使用。
4. 用这个 tex 文档测试编译（⌃⌥B）。

    ```
    \documentclass[UTF8]{article}
    \author {Author}
    \title {Title}
    \usepackage{ctex}
    \usepackage{amsmath}
    \usepackage{amssymb}
    \begin{document}
    \maketitle
    \section{First section} test1.
        \subsection{First subsection} test2.
            \subsubsection{First double subsection}
                \paragraph{Fist paragraph} test3.
                    \subparagraph{First subparagraph} test4.
        \subsection{Second subsection}
            \paragraph{段落} 中文测试。
    \\
    Hello World! \\ % This is comment
    Hello \LaTeX ! \\

    $\lim\limits_{n \rightarrow +\infty} P\lbrace\frac{\sum\limits_{i=1}{n}Xi - n\cdot EX}{ \sqrt{n \cdot DX} }  \leqslant x\rbrace = \Phi(x)$ \\

    $P\lbrace a<X<b \rbrace \approx \Phi(\frac{b - n\cdot EX}{\sqrt {n\cdot DX}}) - \Phi(\frac{a - n\cdot EX}{\sqrt{n\cdot DX} })$ \\

    $F(x,y) = F_{X}(x)F_{Y}(y)$
    \end{document}
    ```

### Aria2GUI

1. 下载并安装 [Aria2GUI](https://github.com/yangshun1029/aria2gui)。
2. 下载 [BaiduExporter](https://github.com/acgotaku/BaiduExporter) 并安装 Chrome 扩展程序。

### GitHub Pages

1. 在 Atom 中安装插件 “Markdown Writer”。
2. 安装 toolbar 插件辅助 Markdown 编辑。
    1. Atom -> Install Shell Commands。
    2. 然后执行 `apm install tool-bar markdown-writer` 来安装 `tool-bar` 和 `markdown-writer` packages。
    3. 接着执行 `apm install tool-bar-markdown-writer`。
3. 注意所使用的模板可能需要另外安装一些依赖。之后才能在本地运行 `jekyll serve`。


### Maven 及其配置

用到的时候补充。

### 其他

- 辅助工具
  - Alfred 3 - Spotlight 的辅助工具
  - The Unarchiver / Archiver - 解压缩工具
  - FruitJuice - 电池管理小插件
  - iStatistica - 系统管理小插件
  - [CheatSheet](https://www.mediaatelier.com/CheatSheet/) - 快捷键显示插件
  - [AppCleaner](http://freemacsoft.net/appcleaner/) - 应用清理工具

- 网络工具
  - Tunnelblick / ShadowsocksX - VPN
  - [uTorrent](http://www.utorrent.com/intl/zh/) - 下载器

- 多媒体
  - [IINA](https://lhc70000.github.io/iina/)（含 Safari 插件、Chrome 扩展程序） - 播放器
  - [Typora](https://www.typora.io) - Markdown 编辑器
