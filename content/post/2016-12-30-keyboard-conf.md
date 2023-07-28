---
layout: post
title: Mac上配置GH60键盘
categories: [生活随想]
tags: [效率工具, Mac, 键盘]
date: 2016-12-30
description: 在Mac上编译GH60 RevCHN Firmware
keywords: Keyboard, Mac, Note
---

### 安装必要的工具

    brew install Caskroom/cask/crosspack-avr
    brew install dfu-programmer
    cd ~
    git clone --recursive https://github.com/kairyu/tmk_keyboard_custom.git

* 检查 `avr-gcc` 命令是否可用

  * 直接在命令行执行 `avr-gcc`, 如果出现command not found, 在命令行执行：

            PATH="/usr/local/CrossPack-AVR/bin:$PATH"
            export $PATH

  * 以上命令是临时把路径 `/usr/local/CrossPack-AVR/bin` 放到 `PATH` 中，关闭Terminal后失效

### 修改配置文件来支持GH60_REV_CHN  

* `cd ./tmk_keyboard_custom/keyboard/gh60`
* `vi config.h`
* 找到 `#define CONFIG_H`, 在底下加上 `#define GH60_REV_CHN`

* `vi Makefile`
* 注释或删掉下面这行代码

        KEYMAP_IN_EEPROM_ENABLE = yes # Read keymap from eeprom

### 连接测试键盘

* 连接后输入入下列指令

        system_profiler SPUSBDataType

* 应该会出现：

```
GH60:
   Product ID: 0x6060
   Vendor ID: 0xfeed
   Version: 0.01
   Speed: Up to 12 Mb/sec
   Manufacturer: geekhack
   Location ID: 0x14400000 / 37
   Current Available (mA): 500
   Current Required (mA): 100
   Extra Operating Current (mA): 0
```

* 然后按下键盘背部的按钮
* 再次执行命令

        system_profiler SPUSBDataType

* 应该会出现：

```
ATm32U4DFU:
    Product ID: 0x2ff4
    Vendor ID: 0x03eb  (Atmel Corporation)
    Version: 0.00
    Serial Number: 1.0.0
    Speed: Up to 12 Mb/sec
    Manufacturer: ATMEL
    Location ID: 0x14100000 / 15
    Current Available (mA): 500
    Current Required (mA): Unknown (Device has not been configured)
```

* 可以开始刷配置了，如果没信心，还是先用 poker 的 layout 来试试吧

        poker layout

* 切换到目录 `gh60` 下执行

        make dfu

* 刷出以下内容表示成功

```
Creating load file for Flash: gh60_lufa.hex
avr-objcopy -O ihex -R .eeprom -R .fuse -R .lock -R .signature gh60_lufa.elf gh60_lufa.hex
dfu-programmer atmega32u4 erase --force
Erasing flash...  Success
Checking memory from 0x0 to 0x6FFF...  Empty.
dfu-programmer atmega32u4 flash gh60_lufa.hex
Checking memory from 0x0 to 0x61FF...  Empty.
0%                            100%  Programming 0x6200 bytes...
[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
0%                            100%  Reading 0x7000 bytes...
[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
Validating...  Success
0x6200 bytes written into 0x7000 bytes memory (87.50%).
dfu-programmer atmega32u4 reset
```

### 开始制作自己的layout

* [http://www.keyboard-layout-editor.com/](http://www.keyboard-layout-editor.com/) 生成raw data
* [https://yang.hging.net/tkg/](https://yang.hging.net/tkg/) 把raw data生成`.c`文件
* 参考[dm4](http://blog.dm4.tw/blog/2015/03/17/build-gh60-revchn-on-mac/)设计的layout，觉得不错稍加改了下:

  * Layer 0 (Default)

            http://www.keyboard-layout-editor.com/#/gists/907d4f327ee92188cb929044ec186c39

  * Layer 1 (Fn0层)

            http://www.keyboard-layout-editor.com/#/gists/a460aca35c9eedab94433f6c297ef94c

  * keymap generator 的设定

            Keyboard - GH60 (RevCHN)
            Layer Mode - Normal
            Number of Layers - 2
            Layer0 - Default
            Layer1 - Function layer
            Fn - Layer action > Momentary layer 1

  * Fn Momentary layer 1 的意思就是当 fn 按下时是 layer 1 的 keymap 但不按 fn 时就关掉 layer1

  * 下载 `.c` 文件, 把名字改成 `keymap_tkg.c`

  * 放到 `gh60` 目录下边

  * 执行 `make KEYMAP=tkg dfu`

### Done

### 记录下现有的问题

* `Fn+RCommand+Delete 连按几下会导致 Command 键按下事件，每次再按其他键的时候都成了Command+键的效果。现在的解决方案就是重新多按几个那个组合。。。。

### Refer to

* <http://blog.dm4.tw/blog/2015/03/17/build-gh60-revchn-on-mac/>
* <https://www.ptt.cc/bbs/Key_Mou_Pad/M.1430970988.A.4D9.html>
