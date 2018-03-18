---
layout: post
title: "重新安装 macOS 的配置"
date: 2017-10-11
tags: [macos]
categories:
  - 日常
  - 备忘
---

本篇文章介绍了每次抹掉系统并重新安装 macOS 必要的恢复操作。这些工作容易遗忘，所以记下来。

<!-- more -->

### iTerm2 + oh-my-zsh

1. 安装 [Homebrew](https://brew.sh) 和 iTerm2。
3. 通过指令安装 Oh My Zsh。
  `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
4. 设置主题为 `af-magic`。
5. 通过 Homebrew 安装插件：
  `brew install zsh-syntax-highlighting`
  `brew install zsh-autosuggestions`

> 参考 [iTerm 2 && Oh My Zsh【DIY教程——亲身体验过程】 - 简书](http://www.jianshu.com/p/7de00c73a2bb) 和 [Mac下终端配置（item2 + oh-my-zsh + solarized配色方案） - 希希里之海 - 博客园](http://www.cnblogs.com/weixuqin/p/7029177.html)。


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
3. 在 latex 插件的配置中选择 xelatex 即可使用。

### Aria2GUI

1. 下载并安装 [Aria2GUI](https://github.com/yangshun1029/aria2gui)。
2. 下载 [BaiduExporter](https://github.com/acgotaku/BaiduExporter) 并加载到 Chrome 扩展程序中。

### GitHub Pages

1. 下载并安装 [Node.js](https://nodejs.org/en/)。
2. 通过指令安装 hexo，可能需要 sudo。
  `npm install -g hexo-cli`
3. 下载对应的主题或已建立的博客项目。
4. 如果没有配置 Git 的用户名、邮箱和设置 SSH，先按照相应步骤设置。

>1. 在 Atom 中安装插件 “Markdown Writer”。
>2. 安装 toolbar 插件辅助 Markdown 编辑。
>    1. Atom -> Install Shell Commands。
>    2. 然后执行 `apm install tool-bar markdown-writer` 来安装 `tool-bar` 和 `markdown-writer` packages。
>    3. 接着执行 `apm install tool-bar-markdown-writer`。
>3. 注意所使用的模板可能需要另外安装一些依赖。之后才能在本地运行 `jekyll serve`。


### Maven 及其配置

用到的时候补充。

### 其他

#### 辅助工具
  - The Unarchiver / Archiver - 解压缩工具
  - FruitJuice - 电池管理小插件
  - iStatistica - 系统管理小插件
  - [CheatSheet](https://www.mediaatelier.com/CheatSheet/) - 快捷键显示插件
  - [AppCleaner](http://freemacsoft.net/appcleaner/) - 应用清理工具

#### 网络工具
  - Tunnelblick / ShadowsocksX - VPN
  - [uTorrent](http://www.utorrent.com/intl/zh/) - 下载器
  - [CyberDuck](https://cyberduck.io) - FTP 工具

#### 多媒体
  - [IINA](https://lhc70000.github.io/iina/)（含 Safari 插件、Chrome 扩展程序） - 播放器

#### 工作
  - [Typora](https://www.typora.io) - Markdown 编辑器
  - [Atom](https://atom.io) - 文本编辑器
