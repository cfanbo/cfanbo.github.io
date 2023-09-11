---
title: Visual Studio Code之常备快捷键
author: admin
type: post
date: 2016-09-11T01:28:58+00:00
url: /archives/17100
categories:
 - 程序开发

---
推荐：[Visual Studio Code 添加设置代码段(snippet)][1]{#cb_post_title_url.postTitle2}

官方快捷键大全： [https://code.visualstudio.com/docs/customization/keybindings](https://code.visualstudio.com/docs/customization/keybindings)

Visual Studio Code是个牛逼的编辑器，启动非常快，完全可以用来代替其他文本文件编辑工具。又可以用来做开发，支持各种语言，相比其他IDE，轻量级完全可配置还集成Git感觉非常的适合前端开发。 所以我仔细研究了一下文档未来可能会作为主力工具使用。

# 主命令框 {#主命令框_Command_Palette}

最重要的功能就是`F1`或`Ctrl+Shift+P`打开的命令面板了，在这个命令框里可以执行VSCode的任何一条命令，甚至关闭这个编辑器。
按一下`Backspace`会进入到`Ctrl+P`模式里
在`Ctrl+P`下输入`>`又可以回到`Ctrl+Shift+P`模式。
在`Ctrl+P`窗口下还可以

 * 直接输入文件名，跳转到文件
 * `?` 列出当前可执行的动作
 * `!` 显示Errors或Warnings，也可以\`Ctrl+Shift+M
 * `:` 跳转到行数，也可以`Ctrl+G`直接进入
 * `@` 跳转到symbol（搜索变量或者函数），也可以`Ctrl+Shift+O`直接进入
 * `@:`根据分类跳转symbol，查找属性或函数，也可以`Ctrl+Shift+O`后输入`:`进入
 * `#` 根据名字查找symbol，也可以`Ctrl+T`

# 常用快捷键 {#常用快捷键}

* * *

## 编辑器与窗口管理 {#编辑器与窗口管理}

### 同时打开多个窗口（查看多个项目） {#同时打开多个窗口（查看多个项目）}

 * 打开一个新窗口： `Ctrl+Shift+N`
 * 关闭窗口： `Ctrl+Shift+W`

### 同时打开多个编辑器（查看多个文件） {#同时打开多个编辑器（查看多个文件）}

 * 新建文件 `Ctrl+N`
 * 文件之间切换 `Ctrl+Tab`
 * 切出一个新的编辑器（最多3个）`Ctrl+\`，也可以按住Ctrl鼠标点击Explorer里的文件名
 * 左中右3个编辑器的快捷键`Ctrl+1` `Ctrl+2` `Ctrl+3`
 * 3个编辑器之间循环切换 Ctrl+\`
 * 编辑器换位置，`Ctrl+k`然后按`Left`或`Right`

## 代码编辑 {#代码编辑}

### 格式调整 {#格式调整}

 * 代码行缩进`Ctrl+[` `Ctrl+]`
 * `Ctrl+C` `Ctrl+V`如果不选中，默认复制或剪切一整行
 * 代码格式化：`Shift+Alt+F`，或`Ctrl+Shift+P`后输入`format code`
 * 上下移动一行： `Alt+Up` 或 `Alt+Down`
 * 向上向下复制一行： `Shift+Alt+Up`或`Shift+Alt+Down`
 * 在当前行下边插入一行`Ctrl+Enter`
 * 在当前行上方插入一行`Ctrl+Shift+Enter`

### 光标相关 {#光标相关}

 * 移动到行首：`Home`
 * 移动到行尾：`End`
 * 移动到文件结尾：`Ctrl+End`
 * 移动到文件开头：`Ctrl+Home`
 * 移动到定义处：`F12`
 * 定义处缩略图：只看一眼而不跳转过去`Alt+F12`
 * 移动到后半个括号 `Ctrl+Shift+]`
 * 选择从光标到行尾`Shift+End`
 * 选择从行首到光标处`Shift+Home`
 * 删除光标右侧的所有字`Ctrl+Delete`
 * Shrink/expand selection： `Shift+Alt+Left`和`Shift+Alt+Right`
 * Multi-Cursor：可以连续选择多处，然后一起修改，`Alt+Click`添加cursor或者`Ctrl+Alt+Down` 或 `Ctrl+Alt+Up`
 * 同时选中所有匹配的`Ctrl+Shift+L`
 * `Ctrl+D`下一个匹配的也被选中(被我自定义成删除当前行了，见下边`Ctrl+Shift+K`)
 * 回退上一个光标操作`Ctrl+U`

### 重构代码 {#重构代码}

 * 找到所有的引用：`Shift+F12`
 * 同时修改本文件中所有匹配的：`Ctrl+F12`
 * 重命名：比如要修改一个方法名，可以选中后按`F2`，输入新的名字，回车，会发现所有的文件都修改过了。
 * 跳转到下一个Error或Warning：当有多个错误时可以按`F8`逐个跳转
 * 查看diff 在explorer里选择文件右键 `Set file to compare`，然后需要对比的文件上右键选择`Compare with 'file_name_you_chose'`.

### 查找替换 {#查找替换}

 * 查找 `Ctrl+F`
 * 查找替换 `Ctrl+H`
 * 整个文件夹中查找 `Ctrl+Shift+F`
 匹配符：
 * `*` to match one or more characters in a path segment
 * `?` to match on one character in a path segment
 * `**` to match any number of path segments ,including none
 * `{}` to group conditions (e.g. `{**/*.html,**/*.txt}` matches all html and txt files)
 * `[]` to declare a range of characters to match (e.g., `example.[0-9]` to match on `example.0`,`example.1`, …

## 显示相关 {#显示相关}

 * 全屏：`F11`
 * zoomIn/zoomOut：`Ctrl + =`/`Ctrl + -`
 * 侧边栏显/隐：`Ctrl+B`
 * 侧边栏4大功能显示：
 * Show Explorer `Ctrl+Shift+E`
 * Show Search`Ctrl+Shift+F`
 * Show Git`Ctrl+Shift+G`
 * Show Debug`Ctrl+Shift+D`
 * Show Output`Ctrl+Shift+U`

## 其他 {#其他}

 * 自动保存：File -> AutoSave ，或者`Ctrl+Shift+P`，输入 auto

# 修改默认快捷键 {#修改默认快捷键}

* * *

File -> Preferences -> Keyboard Shortcuts

修改`keybindings.json`，我的显示在这里`C:\Users\Administrator\AppData\Roaming\Code\User\keybindings.json`


```
// Place your key bindings in this file to overwrite the defaults
[
    //ctrl+space被切换输入法快捷键占用
    {
        "key": "ctrl+alt+space",
        "command": "editor.action.triggerSuggest",
        "when": "editorTextFocus"
    },
    // ctrl+d删除一行
    {
        "key": "ctrl+d",
        "command": "editor.action.deleteLines",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+shift+k", //与删除一行的快捷键互换了：）
        "command": "editor.action.addSelectionToNextFindMatch",
        "when": "editorFocus"
    },
    //ctrl+shift+/多行注释
    {
        "key":"ctrl+shift+/",
        "command": "editor.action.blockComment",
        "when": "editorTextFocus"
    }
]

```



# 插件 {#插件}

新版本支持插件安装了
插件市场 [https://marketplace.visualstudio.com/#VSCode](https://marketplace.visualstudio.com/#VSCode)

## 安装插件 {#安装插件}

`F1` 输入 `extensions`

[![](http://nshen.net/image/vscode/29945359.png)](http://nshen.net/image/vscode/29945359.png "")

点击第一个开始安装或升级，或者也可以  `Ctrl+P` 输入 `ext install`进入
点击第二个会列出已经安装的扩展，可以从中卸载

 [1]: http://www.cnblogs.com/summit7ca/p/5225494.html