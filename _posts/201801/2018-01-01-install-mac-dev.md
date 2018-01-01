---
layout: post
title:  "同步开发环境到一台全新的电脑"
date:   2018-01-01 1:30:00
categories: tool
tags: mac
author: "Victor"
---

用了将近 10 年的 Mac 终于在 2017 年底的倒数第2天买了第一台 MacbookPro。怎么样一边玩游戏一边把电脑开发环境配置好呢？

开机之后先登录 MAS 更新系统补丁，安装Xcode。

## Preferences

* `Keyboard > Text >` Disable "Correct spelling automatically".
* `Security and Privacy > Firewall >` On
* `Security and Privacy > General >` App Store and identified developers
* `File Sharing >` Off
* `Trackpad >` Tap to click
* `Keyboard > Key Repeat >` Fast (all the way to the right)
* `Keyboard > Delay Until Repeat >` Short (all the way to the right)
* `Dock >` Automatically hide and show the Dock

## DropBox

先配好全局跳墙，然后去 DropBox 官网下载客户端并登录同步自己的文件夹。这里有 Mackup 的全部配置。

## Homebrew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果开局已经登录 MAS 下面这步就可以忽略了。

```bash
brew install mas
mas signin email@email.com
```

执行 `brew bundle exec -- bundle install` 搞定全部软件。

[这里](https://gist.github.com/wjp2013/66dbd649203e822eb6da110300fead47) 是我的 Brewfile。

## Mackup

把软件配置都同步过来。

```bash
mackup restore
```

## SpaceVim

```bash
curl -sLf https://spacevim.org/install.sh | bash
```

## Dock && Menu

按照原来的电脑把 Dock 上的图标和 Menu 上的图标都排列好，正版软件都激活。

## iTerm

```bash
# Specify the preferences directory
defaults write com.googlecode.iterm2.plist PrefsCustomFolder -string "~/Dropbox/Apps/iTerm2"
# Tell iTerm2 to use the custom preferences in the directory
defaults write com.googlecode.iterm2.plist LoadPrefsFromCustomFolder -bool true
```

## Other

1. 用 rbenv 安装所需版本的 Ruby
2. Atom `sync-settings:restore` 同步已经安装的插件和配置
3. Alfred `defaults write com.runningwithcrayons.Alfred-Preferences-3 dropbox.allowappsfolder -bool TRUE` 然后修改 General 中的 sync 文件夹到 `~/Dropbox/Apps/Alfred`

## 相关链接

* https://github.com/taniarascia/mac
* https://github.com/nicolashery/mac-dev-setup
* https://mallinson.ca/osx-web-development/
* https://www.taniarascia.com/setting-up-a-brand-new-mac-for-development/
* https://github.com/Homebrew/homebrew-bundle
* https://robots.thoughtbot.com/brewfile-a-gemfile-but-for-homebrew
* https://github.com/sharat/vscode-brewfile