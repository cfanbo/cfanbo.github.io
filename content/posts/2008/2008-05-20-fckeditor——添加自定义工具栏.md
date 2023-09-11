---
title: FCKEditor——添加自定义工具栏
author: admin
type: post
date: 2008-05-20T23:37:25+00:00
excerpt: |
 FCKEditor是一个功能强大的开源在线编辑器，所以是非常适合我等兜兜无啥银子的人拿来“把玩”的~~~呵呵。一个产品即使功能再强大也不能满足所有用户的需求，当然FCKEditor也不例外咯。就拿我现在所开发的一个系统（工作流）来说，就遇到了FCKEditor不能满足我的要求的情况。因为我想在工具栏中加入自己的操作控制按钮，配置当然搞不定咯，就只有改源代码了。可一看FCKEditor经过处理后的JS源码，头立刻就大了——无换行无注释，一大堆JS代码堆在那里，想看懂几乎没门。当然它这样做也是有好处的，要不这大的一个东西加载怎么会那么快呢。看不懂处理后的JS源码，我们可以看有格式的源码嘛，所以就上网down了一个2.4的FCKEditor。好了，现在就让我们开始怎么一步一步的加入我们自己的操作菜单到工具栏中去。

 比如我想加一个我自己的输入框用来控制日期的输入，即该输入框只能通过选择来选择日期，这个我们结合日期控件my97来做，呵呵，充分利用已有的成果。有人会说，我直接改它的对话框不就得了
url: /archives/285
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - FCKEditor

---
   FCKEditor是一个功能强大的开源在线编辑器，所以是非常适合我等兜兜无啥银子的人拿来“把玩”的~~~呵呵。一个产品即使功能再强大也不能满足所有用户的需求，当然FCKEditor也不例外咯。就拿我现在所开发的一个系统（工作流）来说，就遇到了FCKEditor不能满足我的要求的情况。因为我想在工具栏中加入自己的操作控制按钮，配置当然搞不定咯，就只有改源代码了。可一看FCKEditor经过处理后的JS源码，头立刻就大了——无换行无注释，一大堆JS代码堆在那里，想看懂几乎没门。当然它这样做也是有好处的，要不这大的一个东西加载怎么会那么快呢。看不懂处理后的JS源码，我们可以看有格式的源码嘛，所以就上网down了一个2.4的FCKEditor。好了，现在就让我们开始怎么一步一步的加入我们自己的操作菜单到工具栏中去。

    比如我想加一个我自己的输入框用来控制日期的输入，即该输入框只能通过选择来选择日期，这个我们结合日期控件my97来做，呵呵，充分利用已有的成果。有人会说，我直接改它的对话框不就得了，当然这也是可以的，可我们今天要做的就是如何加入自己的工具栏操作，这样以后就可以依次类推，加入任何想要加入的操作了:-)。

    首先，我们给这个日期输入框定一个名字，就叫做CTRL\_Date吧。第一步嘛，我们就直接在fckconfig.js文件中的Basic工具条集（FCKConfig.ToolbarSets[“Basic”]）中加入这个名称，加进去很简单吧，呵，不多说了。然后呢，也得给它弄个中文名称什么的吧，直接打开语言文件zh-cn.js，英文的就不管了哈，加入：CTRL\_Date : “日期控件”。另外还得加一个上下文语言提示菜单，这样在编辑区域内就可以通过右键选择属性来进行编辑操作了。也是在zh-ch.js中加入：CTRLDateProp : “日期控件属性”。到这里，又要回到fckconfig.js文件中了，我们在FCKConfig.ContextMenu中加入CTRL\_Date，这样才能使其拥有上下文菜单嘛。到这里，基础工作就做完了，呵呵，然后就是艰巨的源码改造工程了，当然了也不要怕咯，一步一步来就没问题了~~@\_@

    我们解压下载的FCKEditor2.4，然后找到fckregexlib.js文件，找到其中的NamedCommands，然后把CTRL\_Date加到最后，然后再修改fckeditor处理后的最终JS文件fckeditorcode\_ie.js（只改IE的，至于FF就不管了，对不起它了，嘿）。网上说可以用一个什么程序来进行源码打包的，可我在下的压缩包里没看到呢，所以就只有手动修改这个最终的文件了。在该文件中找到NamedCommands这一位置然后在最后加入CTRL\_Date，这样就注册了一个命令CTRL\_Date了。然后再找到源代码文件夹中的fckcommands.js文件，是不是看到很多case啊，呵，这就是工具栏中每个操作的命令了。我们依葫芦画瓢，在最后加上：case ‘CTRL\_Date’:oCommand=new FCKDialogCommand(‘CTRL\_Date’,FCKLang.CTRL\_Date,’dialog/workflow/fck\_ctrl\_date.html’,380,230);break ;这个意思就是说，当你单击工具栏上的日期控件的时候就会以模式对话框打开dialog/workflow/fck\_ctrl\_date.html这个网页文件，至于是什么内容，我们待会再说。这样写好后，我们就可以加入到fckeditorcode\_ie.js文件中了。我们在fckeditorcode_ie.js文件中找到new FCKUndefinedCommand()这个位置，然后在它之后的break;之后加上我们刚才写的代码（注意把oCommand改为B后再加入，因为我的最终处理源码中是B，估计这样简写也是为了减少源码的大小吧）。这两个步骤完成后，我们就可以在工具栏中单击日期控件时打开我们自己定义的日期控件属性编辑的网页文件了。

    然后还一工程就是增加该控件的上下文菜单，要不然在编辑区域右键选择时就会弹出输入框的属性和对话框了，这可不是我们想要的结果呢。我们还是先在源码中修改，然后再复制到最终处理的源码文件中去。找到文件fcktoolbaritems.js，也有很多case哦，看名字就知道这是设置工具栏中的按钮项了。同样，我们在最后加入：case ‘CTRL\_Date’ : oItem=new FCKToolbarButton(‘CTRL\_Date’,FCKLang.CTRL\_Date,null,null,null,null,51);break;注意其中的55表示的是该操作按钮的图标索引（这个索引指的是skins目录下的fck\_strip.gif文件中图标的顺序索引），我们使用和输入框一样的图标就是51了。同样，我们在最终文件fckeditorcode_ie.js中查找字符串alert(FCKLang.UnknownToolbarItem.replace(/%1/g（注意把空格去掉），然后在它之前的default之前加入我们刚写的语句（同样把oItem改成B）。这是完成上下文菜单的第一步，还有第二步也就是最后一步了，胜利就在眼前哦，加油了，呵呵。

    接着就是文件fck_contextmenu.js的修改了。一样也是很多case，我们加入如下语句
case ‘CTRL\_Date’:return{AddItems:function(A,B,C){if(C==’INPUT’&&B.type==’text’&&B.className==’\_wf\_date’){A.AddSeparator();A.AddItem(‘CTRL\_Input’,FCKLang.CTRLDateProp,51);}}};（注意这里的A，B，C就是源码中menu，tag，tagName，这里直接写成A，B，C是为了直接插入到最终代码中去而已）。注意到上面中不是有个B.className==’\_wf\_date’么？这是因为FCKEditor中已经自带了input输入框的控制，所以这里我们用一个样式名称来区别它自带的还是我们自加的。当然这个样式名称可以随便取的，而且在fck\_ctrl\_date.html这个网页文件（我们自己写的来编辑日期控件的属性文件）中需要给日期控件也就是input输入框加上className=”\_wf\_date”属性以示区别原有的input输入框。当然，我们还要修改原来的input的case，在fckeditorcode\_ie.js文件中找到字符串case ‘TextField’:return {AddItems所在的位置（如果找不到请去掉或增加空格），然后修改if中的条件，加一个&&B.className!=’\_wf_date’。最后就把我们写的上面的代码加到TextField这个case之后就可以了。这样修改后就会使当右键单击某个元素时，如果是输入框并且样式名称不是我们指定的日期控件则会调用它自带的输入框属性编辑网页，如果样式名称是我们自己定义的日期控件的名称则会调用我们自己编辑的属性网页，这样就达到了同是输入框在查看属性时分别调用不同属性编辑网页的目的了。

    好了，完成上面的所有步骤后就可以保存修改后的fckeditorcode\_ie.js文件了。这里还有一点工作要做哦，就是在dialog目录下新建一个目录workflow，然后在它下新建一个日期控件属性输入的网页文件fck\_ctrl\_date.html。具体的内容我们可以直接copy FCKEditor自带的fck\_textfield.html网页文件的内容过来，然后简单的修改下就可以了，但重要的是别忘了在ok函数中给你的日期控件加上className=”\_wf\_date”属性哦，否则右键属性查看时则会调用FCKEditor自带的文本框输入的编辑网页了。

    经过这么多步骤，已经全部完工了。赶快打开你的在线编辑器看看吧~~~~