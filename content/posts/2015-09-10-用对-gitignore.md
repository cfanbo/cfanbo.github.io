---
title: git中 gitignore 文件的正确用法
author: admin
type: post
date: 2015-09-10T04:52:01+00:00
url: /archives/15965
categories:
 - 程序开发
tags:
 - git
 - gitignore

---
使用 git 做代码管理工具时，设置 gitignore 是必不可少的流程，一些系统或者 IDE 会在目录下生成与项目不相关的文件，而这些文件我们不期望被提交到仓库之中。理解 gitignore 的 pattern 规则十分重要。

### **Pattern 规则** {#Pattern_规则}

关于 Pattern 规则，可以查看 git 的相关文档： [http://git-scm.com/docs/gitignore](http://git-scm.com/docs/gitignore)，大致有以下几点：

 1. 空行不匹配任何内容，所以可以作为块分隔符；
 2. `#` 开头表示注释，如果相匹配 `#`，可以在前面加一个反斜杠，即 `\#`；
 3. 除非加了反斜杠，否则一连串的空格会被忽略；
 4. 如果在匹配的内容前面加上 `!`，则这些匹配过的部分将被移出，如果要匹配以 `!` 开头的内容，需要加上反斜杠，如 `\!important.txt`；
 5. 如果一个匹配 pattern 后面有一个斜杠，如 `foo/`，则默认会匹配所有（包含父子文件夹）中的 foo 文件夹内容，并且它不会匹配单个的文件；
 6. 如果一个匹配 pattern 不包含斜杠，如 `foo`，Git 会将其作为一个 shell 的查找命令匹配内容。

需要注意的`**`：

 * 如果一个 pattern 以 `**` 开头，如 `**/foo`，最后会匹配所有文件夹下的 foo 文件(夹)；
 * 如果一个 pattern 以 `/**` 开头，如 `abc/**`，则表示匹配 abc 目录下的所有内容（(relative to the location of the `.gitignore` file, with infinite depth)；
 * 如果一个 pattern 中间包含 `**`，如 `a/**/b`，则会匹配 `a/b`、`a/x/b`、`a/x/y/b`以及所有类似的内容。

### **gitignore 相关的问题** {#gitignore_相关的问题}

#### 匹配示例 {#匹配示例}

如果我们要匹配 ‘foo’ 目录下除去 ‘foo/bar/‘ 的内容，可以这样做：

```
foo/
!foo/bar/
```

如果要匹配所有目录下的 node_modules 文件夹，只需要这样做：

```
node_modules/
```

如果要匹配所有的 json 文件，可以这样做：

```
*.json
```

#### git 操作中，add 之后再加入 gitignore {#git_操作中，add_之后再加入_gitignore}

Git 操作中经常会出现这样的问题，当我们 `git add` 之后，突然想起来要添加一个 gitignore 文件，但是一些诸如 node_modules/, cache/ 等文件已经被 add 进去了，这些文件不会被 ignore 掉，怎么办？

最直接的方式是：

```
# 这一步的操作相当于回到 git add 上一步
git rm -r --cached .
# 然后重新 add
git add  .
git commit -m "fixed untracked files"

```

#### git 添加空文件夹 {#git_添加空文件夹}

Git 默认是不添加空文件夹的，如果一定要加入这个文件夹，有以下方案：

1）在文件夹添加文件，然后删除

2）在文件夹中添加一个 `.gitkeep` 文件

#### 让 git 不要添加 gitignore 文件 {#让_git_不要添加_gitignore_文件}

如果在 .gitignore 文件中添加

```
.gitignore
```

你会发现，并没有起作用， .gitignore 文件依然被加到了 git 中，为什么会有这个需求呢？有些人在本地开发的时候有一些其他的文件夹名不愿意让别人看到，虽然在 gitignore 中被忽略了，但是 .gitignore 文件中依然可以看到这些文件夹名字。

其实没有什么好的办法处理这个问题，.gitignore 做多人协作开发的时候可以直接根据同一份 gitignore 过滤文件，如果一定要做，可以从 add 中在 remove 掉：

```
git rm -r --cached .gitignore
```

Git 操作涉及的命令巨多，但是日常开发中用到的就那么几个，把原理搞清楚，用起来得心应手。

本文链接： [http://www.barretlee.com/blog/2015/09/06/set-gitignore-after-add-file](http://www.barretlee.com/blog/2015/09/06/set-gitignore-after-add-file/)

=======================

    在git中如果想忽略掉某个文件，不让这个文件提交到版本库中，可以使用修改 .gitignore 文件的方法。这个文件每一行保存了一个匹配的规则例如：

```
# 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt

```

另外 git 提供了一个全局的 .gitignore，你可以在你的用户目录下创建 ~/.gitignoreglobal 文件，以同样的规则来划定哪些文件是不需要版本控制的。

需要执行 git config –global core.excludesfile ~/.gitignoreglobal 来使得它生效。

其他的一些过滤条件

```
* ？：代表任意的一个字符
* ＊：代表任意数目的字符
* {!ab}：必须不是此类型
* {ab,bb,cx}：代表ab,bb,cx中任一类型即可
* [abc]：代表a,b,c中任一字符即可
* [ ^abc]：代表必须不是a,b,c中任一字符

```

由于git不会加入空目录，所以下面做法会导致tmp不会存在

```
tmp/*             //忽略tmp文件夹所有文件

```

所以修改使用下面的方法就可以实现。

在tmp下也加一个.gitignore,内容为

```
 *
 !.gitignore
```

这样就可以实现提交一个目录和.gitignore文件了。

有时候为了禁止目录索引，会加入一个index.html 文件，所以还为了防止一些临时目录或者上传目录，还需要忽略掉**index.html**，最后内容下如：

```
 *
!.index.html
!.gitignore
```

还有一种情况，就是已经commit了，再加入gitignore是无效的，所以需要删除下缓存

**git rm -r –cached ignore_file**

注意： .gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

     正确的做法是在每个clone下来的仓库中手动设置不要检查特定文件的更改情况。

git update-index –assume-unchanged PATH    在PATH处输入要忽略的文件。

另外 git 还提供了另一种 exclude 的方式来做同样的事情，不同的是 .gitignore 这个文件本身会提交到版本库中去。用来保存的是公共的需要排除的文件。而 .git/info/exclude 这里设置的则是你自己本地需要排除的文件。 他不会影响到其他人。也不会提交到版本库中去。

**空.gitignore文件提交空目录：****.gitignore 还有个有意思的小功能， 一个空的 .gitignore 文件 可以当作是一个 placeholder 。****当你需要为项目创建一个空的 log 目录时， 这就变的很有用。 你可以创建一个 log 目录 在里面放置一个空的 .gitignore 文件。这样当你 clone 这个 repo 的时候 git 会自动的创建好一个空的 log 目录了。这样也解决了上面 **git 添加空文件夹 **的问题。**