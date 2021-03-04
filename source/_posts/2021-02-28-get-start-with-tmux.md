---
layout: post
title: 如何使用 Tmux
category: 编程学习
tags: [Linux, Tmux]
date: 2021-02-28
description: 如何使用 Tmux
keywords: tmux, linux
---

## Tmux 是什么？

Tmux 是终端的多窗口工具，跟 screen 的功能差不多，能够将当前所有窗口的状态保存到session (会话)里并挂起，之后可以用 reattach session 的方式重新恢复。不同于 screen 的是，tmux能够在一个窗口添加多个面板。比较常用的场景比如，开两个面板，一个打开 vim， 一个用来运行命令；再比如，需要同时操作多个服务器的时候，打开多个面板，同步输入到所有面板上去。这样就能在多个服务器上同时执行同一条命令。

* 安装：
    * Ubuntu & Debian: `sudo apt install tmux`
    * CentOS: `sudo yum install tmux`
    * macOS: `brew install tmux`
        * 需要先安装 [HomeBrew](https://brew.sh/index_zh-cn)

## 什么是 session 以及如何管理 session ？

* 在终端输入`tmux` 后，窗口会自动创建一个session 。所有的操作都会在这个 session 中进行。比如创建多个面板，在面板之间切换等等。session 可以被保存和恢复。比如你在一个 session 中打开了很多窗口，但是需要重启电脑，这个时候就可把 session 保存起来，然后重启之后恢复 session 来继续工作。

* 新建 session
    * `tmux` 或者 `tmux new`
    * `tmux new -s <session_name> [-n <window_name]` 指定 session 的名字和窗口的名字
* 离开 session
    * `Ctrl+b d`: 挂起session，可以随时恢复
* 恢复 session
    * `tmux a`: 恢复最近的session
    * `tmux a -t <session_name>`: 恢复制定的session
* 删除 session
    * 删除当前的session 只需要 exit 即可
    * 删除挂起的session: `tmux kill-session -t <session_name>`

## 常用的管理窗口/面板快捷键

刚开始使用tmux的时候，快捷键有点不太适应。比如`Ctrl+b c`，我们正常情况下是按下`Ctrl`和`b`的同时按`c`。但是在tmux的session里，是按下`Ctrl+b`松开，然后按c。`Ctrl+b`用来激活命令模式。（以下用 `<prefix>` 代替 `Ctrl+b`）

* `<prefix> c`: 创建新的窗口
* `<prefix> w`: 打开窗口列表
* `<prefix> 1`: 切换至窗口 1 （或任意数字）
* `<prefix> ,`: 重命名当前窗口
* `<prefix> %`: 水平切分当前面板
* `<prefix> "`: 垂直切分当前面板
* `<prefix> o`: 切换到下一个面板
* `<prefix> ;`: 在上一个和下一个面板之间切换
* `<prefix> x`: 关闭当前面板

说实话，我并没有记住所有的快捷键。对我来说，这些快捷键都太奇怪了。所以我做了很多自定义配置。

## 其他更有趣的用法

`.tmux.conf` 是tmux的配置文件，默认放在`~`目录下。Github上有很多人分享自己的配置。我大概搜了下做了下调整，写了个自己的：https://github.com/erictt/dotfiles/blob/master/tmux/.tmux.conf

* 激活快捷键修改成了`Ctrl+a`
    ```
    unbind C-b
    set -g prefix C-a
    bind a send-prefix
    ```
* visual mode里用`v`来选中文字， `y`用来复制。比较可惜的是，不知道怎么把`p`配置成粘贴。好像和默认快捷键有冲突。

    ```
    # Use vi mode to copy
    bind-key -T copy-mode-vi v send-keys -X begin-selection
    bind-key -T copy-mode-vi y send-keys -X copy-selection
    bind-key -T copy-mode-vi r send-keys -X rectangle-toggle
    ```

* `<prefix> + e` 同时操控所有pane, `<prefix> + E` 取消

    ```
    # easily toggle synchronization (mnemonic: e is for echo)
    # sends input to all panes in a given window.
    bind e setw synchronize-panes on
    bind E setw synchronize-panes off
    ```

* `<prefix> + v` 垂直切分窗口，`<prefix> + s` 水平切分窗口

    ```
    # Create splits and vertical splits
    bind-key v split-window -h -p 50 -c "#{pane_current_path}"
    bind-key ^V split-window -h -p 50 -c "#{pane_current_path}"
    bind-key s split-window -p 50 -c "#{pane_current_path}"
    bind-key ^S split-window -p 50 -c "#{pane_current_path}"
    ```    

* 用鼠标来调整窗口大小、切换window/pane

    ```
    # mouse behavior
    set-window-option -g mouse on
    # Using set -gq instead of set -g will silence the unknown option error, while still setting it on old versions of tmux
    setw -gq mode-mouse on
    set -gq mouse-select-pane on
    set -gq mouse-resize-pane on
    set -gq mouse-select-window on
    ```

* 插件 `christoomey/vim-tmux-navigator`。可以直接使用快捷键`Ctrl + hjkl` 来切换pane

## CheatSheet

忘记按键的话，可以在这个网站上搜索。

* https://tmuxcheatsheet.com/

## 参考

* https://github.com/tmux/tmux/wiki/Getting-Started
* https://linuxize.com/post/getting-started-with-tmux/
