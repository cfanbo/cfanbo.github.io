---
title: PHP autoload机制详解
author: admin
type: post
date: 2010-11-08T04:14:13+00:00
url: /archives/6562
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---

**(1) autoload机制概述**

在使用PHP的OO模式开发系统时，通常大家习惯上将每个类的实现都存放在一个单独的文件里，这样会很容易实现对类进行复用，同时将来维护时也很便利。这也是OO设计的基本思想之一。在PHP5之前，如果需要使用一个类，只需要直接使用include/require将其包含进来即可。下面是一个实际的例子：

>

> /* Person.class.php */
>

>
>

>

>
>

> class Person {
>

>
>

> var $name, $age;
>

>
>

> function __construct ($name, $age)
>

>
>

> {
>

>
>

> $this->name = $name;
>

>
>

> $this->age = $age;
>

>
>

> }
>

>
>

> }
>

>
>

> ?>
>

>
>

> /* no_autoload.php */
>

>
>

>

>
>

> require_once (”Person.class.php”);
>

>
>

> $person = new Person(”Altair”, 6);
>

>
>

> var_dump ($person);
>

>
>

> ?>
>

在这个例子中，no-autoload.php文件需要使用Person类，它使用了require_once将其包含，然后就可以直接使用Person类来实例化一个对象。

但随着项目规模的不断扩大，使用这种方式会带来一些隐含的问题：如果一个PHP文件需要使用很多其它类，那么就需要很多的require/include语句，这样有可能会造成遗漏或者包含进不必要的类文件。如果大量的文件都需要使用其它的类，那么要保证每个文件都包含正确的类文件肯定是一个噩梦。

PHP5为这个问题提供了一个解决方案，这就是类的自动装载(autoload)机制。autoload机制可以使得PHP程序有可能在使用类时才自动包含类文件，而不是一开始就将所有的类文件include进来，这种机制也称为lazy loading。

下面是使用autoload机制加载Person类的例子：

>

> /* autoload.php */
>

>
>

>

>
>

> function __autoload($classname) {
>

>
>

> require_once ($classname . “class.php”);
>

>
>

> }
>

>
>

> $person = new Person(”Altair”, 6);
>

>
>

> var_dump ($person);
>

>
>

> ?>
>

通常PHP5在使用一个类时，如果发现这个类没有加载，就会自动运行__autoload()函数，在这个函数中我们可以加载需要使用的类。在我们这个简单的例子中，我们直接将类名加上扩展名”.class.php”构成了类文件名，然后使用require_once将其加载。从这个例子中，我们可以看出autoload至少要做三件事情，第一件事是根据类名确定类文件名，第二件事是确定类文件所在的磁盘路径(在我们的例子是最简单的情况，类与调用它们的PHP程序文件在同一个文件夹下)，第三件事是将类从磁盘文件中加载到系统中。第三步最简单，只需要使用include/require即可。要实现第一步，第二步的功能，必须在开发时约定类名与磁盘文件的映射方法，只有这样我们才能根据类名找到它对应的磁盘文件。

因此，当有大量的类文件要包含的时候，我们只要确定相应的规则，然后在__autoload()函数中，将类名与实际的磁盘文件对应起来，就可以实现lazy loading的效果。从这里我们也可以看出__autoload()函数的实现中最重要的是类名与实际的磁盘文件映射规则的实现。

但现在问题来了，如果在一个系统的实现中，如果需要使用很多其它的类库，这些类库可能是由不同的开发人员编写的，其类名与实际的磁盘文件的映射规则不尽相同。这时如果要实现类库文件的自动加载，就必须在__autoload()函数中将所有的映射规则全部实现，这样的话__autoload()函数有可能会非常复杂，甚至无法实现。最后可能会导致__autoload()函数十分臃肿，这时即便能够实现，也会给将来的维护和系统效率带来很大的负面影响。在这种情况下，难道就没有更简单清晰的解决办法了吧？答案当然是：NO! 在看进一步的解决方法之前，我们先来看一下PHP中的autoload机制是如何实现的。

**(2) PHP的autoload机制的实现**

我们知道，PHP文件的执行分为两个独立的过程，第一步是将PHP文件编译成普通称之为OPCODE的字节码序列（实际上是编译成一个叫做zend_op_array的字节数组），第二步是由一个虚拟机来执行这些OPCODE。PHP的所有行为都是由这些OPCODE来实现的。因此，为了研究PHP中autoload的实现机制，我们将autoload.php文件编译成opcode，然后根据这些OPCODE来研究PHP在这过程中都做了些什么：

>

> /* autoload.php 编译后的OPCODE列表，是使用作者开发的OPDUMP工具
>

>
>

> * 生成的结果，可以到网站 http://www.phpinternals.com/ 下载该软件。
>

>
>

> */
>

>
>

> 1:

>
>

> 2:  // require_once (”Person.php”);
>

>
>

> 3:
>

>
>

> 4:  function __autoload ($classname) {
>

>
>

> 0  NOP
>

>
>

> 0  RECV                1
>

>
>

> 5:   if (!class_exists($classname)) {
>

>
>

> 1  SEND_VAR            !0
>

>
>

> 2  DO_FCALL            ‘class_exists’ [extval:1]
>

>
>

> 3  BOOL_NOT            $0 =>RES[~1]
>

>
>

> 4  JMPZ                ~1, ->8
>

>
>

> 6:    require_once ($classname. “.class.php”);
>

>
>

> 5  CONCAT              !0, ‘.class.php’ =>RES[~2]
>

>
>

> 6  INCLUDE_OR_EVAL     ~2, REQUIRE_ONCE
>

>
>

> 7:   }
>

>
>

> 7  JMP                 ->8
>

>
>

> 8:  }
>

>
>

> 8  RETURN              null
>

>
>

> 9:
>

>
>

> 10:  $p = new Person(’Fred’, 35);
>

>
>

> 1  FETCH_CLASS         ‘Person’ =>RES[:0]
>

>
>

> 2  NEW                 :0 =>RES[$1]
>

>
>

> 3  SEND_VAL            ‘Fred’
>

>
>

> 4  SEND_VAL            35
>

>
>

> 5  DO_FCALL_BY_NAME     [extval:2]
>

>
>

> 6  ASSIGN              !0, $1
>

>
>

> 11:
>

>
>

> 12:  var_dump ($p);
>

>
>

> 7  SEND_VAR            !0
>

>
>

> 8  DO_FCALL            ‘var_dump’ [extval:1]
>

>
>

> 13: ?>
>

在autoload.php的第10行代码中我们需要为类Person实例化一个对象。因此autoload机制一定会在该行编译后的opcode中有所体现。从上面的第10行代码生成的OPCODE中我们知道，在实例化对象Person时，首先要执行FETCH_CLASS指令。我们就从PHP对FETCH_CLASS指令的处理过程开始我们的探索之旅。

通过查阅PHP的源代码(我使用的是PHP 5.3alpha2版本)可以发现如下的调用序列：

ZEND_VM_HANDLER(109, ZEND_FETCH_CLASS, …) (zend_vm_def.h 1864行)

=> zend_fetch_class (zend_execute_API.c 1434行)

=>zend_lookup_class_ex (zend_execute_API.c 964行)

=> zend_call_function(&fcall_info, &fcall_cache) (zend_execute_API.c 1040行)

在最后一步的调用之前，我们先看一下调用时的关键参数：

/* 设置autoload_function变量值为”__autoload” */

fcall_info.function_name = &autoload_function;  // Ooops, 终于发现”__autoload”了

…

fcall_cache.function_handler = EG(autoload_func); // autoload_func !

zend_call_function是Zend Engine中最重要的函数之一，其主要功能是执行用户在PHP程序中自定义的函数或者PHP本身的库函数。zend_call_function有两个重要的指针形参数fcall_info, fcall_cache，它们分别指向两个重要的结构，一个是zend_fcall_info, 另一个是zend_fcall_info_cache。zend_call_function主要工作流程如下：如果fcall_cache.function_handler指针为NULL，则尝试查找函数名为fcall_info.function_name的函数，如果存在的话，则执行之；如果fcall_cache.function_handler不为NULL，则直接执行fcall_cache.function_handler指向的函数。

现在我们清楚了，PHP在实例化一个对象时（实际上在实现接口，使用类常数或类中的静态变量，调用类中的静态方法时都会如此），首先会在系统中查找该类（或接口）是否存在，如果不存在的话就尝试使用autoload机制来加载该类。而autoload机制的主要执行过程为：

(1) 检查执行器全局变量函数指针autoload_func是否为NULL。

(2) 如果autoload_func==NULL, 则查找系统中是否定义有__autoload()函数，如果没有，则报告错误并退出。

(3) 如果定义了__autoload()函数，则执行__autoload()尝试加载类，并返回加载结果。

(4) 如果autoload_func不为NULL，则直接执行autoload_func指针指向的函数用来加载类。注意此时并不检查__autoload()函数是否定义。

真相终于大白，PHP提供了两种方法来实现自动装载机制，一种我们前面已经提到过，是使用用户定义的__autoload()函数，这通常在PHP源程序中来实现；另外一种就是设计一个函数，将autoload_func指针指向它，这通常使用C语言在PHP扩展中实现。如果既实现了__autoload()函数，又实现了autoload_func(将autoload_func指向某一PHP函数)，那么只执行autoload_func函数。

**(3) SPL autoload机制的实现**

SPL是Standard PHP Library(标准PHP库)的缩写。它是PHP5引入的一个扩展库，其主要功能包括autoload机制的实现及包括各种Iterator接口或类。SPL autoload机制的实现是通过将函数指针autoload_func指向自己实现的具有自动装载功能的函数来实现的。SPL有两个不同的函数spl_autoload, spl_autoload_call，通过将autoload_func指向这两个不同的函数地址来实现不同的自动加载机制。

spl_autoload是SPL实现的默认的自动加载函数，它的功能比较简单。它可以接收两个参数，第一个参数是$class_name，表示类名，第二个参数$file_extensions是可选的，表示类文件的扩展名，可以在$file_extensions中指定多个扩展名，护展名之间用分号隔开即可；如果不指定的话，它将使用默认的扩展名.inc或.php。spl_autoload首先将$class_name变为小写，然后在所有的include path中搜索$class_name.inc或$class_name.php文件(如果不指定$file_extensions参数的话)，如果找到，就加载该类文件。你可以手动使用spl_autoload(”Person”, “.class.php”)来加载Person类。实际上，它跟require/include差不多，不同的它可以指定多个扩展名。

怎样让spl_autoload自动起作用呢，也就是将autoload_func指向spl_autoload？答案是使用spl_autoload_register函数。在PHP脚本中第一次调用spl_autoload_register()时不使用任何参数，就可以将autoload_func指向spl_autoload。

通过上面的说明我们知道，spl_autoload的功能比较简单，而且它是在SPL扩展中实现的，我们无法扩充它的功能。如果想实现自己的更灵活的自动加载机制怎么办呢？这时，spl_autoload_call函数闪亮登场了。

我们先看一下spl_autoload_call的实现有何奇妙之处。在SPL模块内部，有一个全局变量autoload_functions，它本质上是一个HashTable，不过我们可以将其简单的看作一个链表，链表中的每一个元素都是一个函数指针,指向一个具有自动加载类功能的函数。spl_autoload_call本身的实现很简单，只是简单的按顺序执行这个链表中每个函数，在每个函数执行完成后都判断一次需要的类是否已经加载，如果加载成功就直接返回，不再继续执行链表中的其它函数。如果这个链表中所有的函数都执行完成后类还没有加载，spl_autoload_call就直接退出，并不向用户报告错误。因此，使用了autoload机制，并不能保证类就一定能正确的自动加载，关键还是要看你的自动加载函数如何实现。

那么自动加载函数链表autoload_functions是谁来维护呢？就是前面提到的spl_autoload_register函数。它可以将用户定义的自动加载函数注册到这个链表中，并将autoload_func函数指针指向spl_autoload_call函数（注意有一种情况例外，具体是哪种情况留给大家思考）。我们也可以通过spl_autoload_unregister函数将已经注册的函数从autoload_functions链表中删除。

上节说过，当autoload_func指针非空时，就不会自动执行__autoload()函数了，现在autoload_func已经指向了spl_autoload_call，如果我们还想让__autoload()函数起作用应该怎么办呢？当然还是使用spl_autoload_register(__autoload)调用将它注册到autoload_functions链表中。

现在回到第一节最后的问题，我们有了解决方案：根据每个类库不同的命名机制实现各自的自动加载函数，然后使用spl_autoload_register分别将其注册到SPL自动加载函数队列中就可了。这样我们就不用维护一个非常复杂的__autoload函数了。

**(4) autoload效率问题及对策**

使用autoload机制时，很多人的第一反应就是使用autoload会降低系统效率，甚至有人干脆提议为了效率不要使用autoload。在我们了解了autoload实现的原理后，我们知道autoload机制本身并不是影响系统效率的原因，甚至它还有可能提高系统效率，因为它不会将不需要的类加载到系统中。

那么为什么很多人都有一个使用autoload会降低系统效率的印象呢？实际上，影响autoload机制效率本身恰恰是用户设计的自动加载函数。如果它不能高效的将类名与实际的磁盘文件(注意，这里指实际的磁盘文件，而不仅仅是文件名)对应起来，系统将不得不做大量的文件是否存在(需要在每个include path中包含的路径中去寻找)的判断，而判断文件是否存在需要做磁盘I/O操作，众所周知磁盘I/O操作的效率很低，因此这才是使得autoload机制效率降低的罪魁祸首!

因此，我们在系统设计时，需要定义一套清晰的将类名与实际磁盘文件映射的机制。这个规则越简单越明确，autoload机制的效率就越高。

结论：autoload机制并不是天然的效率低下，只有滥用autoload，设计不好的自动装载函数才会导致其效率的降低。