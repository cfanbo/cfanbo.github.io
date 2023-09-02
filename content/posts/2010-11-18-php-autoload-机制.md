---
title: PHP autoload 机制
author: admin
type: post
date: 2010-11-18T05:37:53+00:00
url: /archives/6696
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
**1****、简介******

PHP5中引入了类的自动装载(autoload)机制。autoload机制可以使得PHP程序有可能在使用类时才自动包含类文件，而不是一开始就将所有的类文件include进来，这种机制也称为lazy loading。

**例子：******

> /\* autoload.php \*/
>  function __autoload($classname) {
> require_once ($classname . “class.php”);
> }
> $person = new Person(”Altair”, 6);
> var_dump ($person);

通常PHP5在使用一个类时，如果发现这个类没有加载，就会自动运行__autoload()函数，在这个函数中我们可以加载需要使用的类。

autoload至少要做三件事情，**第一件事是根据类名确定类文件名**，**第二件事是确定类文件所在的磁盘路径**(在我们的例子是最简单的情况，类与调用它们的 PHP程序文件在同一个文件夹下)，**第三件事是将类从磁盘文件中加载到系统中**。第三步最简单，只需要使用include/require即可。要实现第一 步，第二步的功能，**必须在开发时约定类名与磁盘文件的映射方法**，只有这样我们才能根据类名找到它对应的磁盘文件。

当有大量的类文件要包含的时候，我们只要确定相应的规则，然后在\_\_autoload()函数中，将类名与实际的磁盘文件对应起来，就可以实现lazy loading的效果。从这里我们也可以看出\_\_autoload()函数的实现中最重要的是类名与实际的磁盘文件映射规则的实现。

存在的问题：如果在一个系统的实现中，如果需要使用很多其它的类库，这些类库可能是由不同的开发人员编写的，其类名与实际的磁盘文件的映射规则不尽相同。这时如果要实 现类库文件的自动加载，就必须在\_\_autoload()函数中将所有的映射规则全部实现，这样的话\_\_autoload()函数有可能会非常复杂，甚至 无法实现。最后可能会导致__autoload()函数十分臃肿，这时即便能够实现，也会给将来的维护和系统效率带来很大的负面影响。

**2****、****PHP****的****autoload****机制的实现******

PHP在实例化一个对象时（实际上在实现接口，使用类常数或类中的静态变量，调用类中的静态方法时都会如此），首先会在系统中查找该类（或接口）是否存在，如果不存在的话就尝试使用autoload机制来加载该类。而autoload机制的主要执行过程为：

(1) 检查执行器全局变量函数指针autoload_func是否为NULL。
(2) 如果autoload\_func==NULL, 则查找系统中是否定义有\__autoload()函数，如果没有，则报告错误并退出。
(3) 如果定义了\_\_autoload()函数，则执行\_\_autoload()尝试加载类，并返回加载结果。
(4) 如果autoload\_func不为NULL，则直接执行autoload\_func指针指向的函数用来加载类。注意此时并不检查__autoload()函数是否定义。

PHP提供了两种方法来实现自动装载机制，一种我们前面已经提到过，是使用用户定义的__autoload()函数，**这通常在****PHP****源程序中来实现**；另外 一种就是设计一个函数，将autoload_func指针指向它，**这通常使用****C****语言在****PHP****扩展中实现**。如果既实现了_\_autoload()函数，又实 现了autoload\_func(将autoload\_func指向某一PHP函数)，那么只执行autoload\_func函数。

**3****、****SPL autoload****机制的实现******

SPL是Standard PHP Library(标准PHP库)的缩写。它是PHP5引入的一个扩展库，其主要功能包括autoload机制的实现及包括各种Iterator接口或类。 SPL autoload机制的实现是通过将函数指针autoload\_func指向自己实现的具有自动装载功能的函数来实现的。SPL有两个不同的函数 spl\_autoload, spl\_autoload\_call，通过将autoload_func指向这两个不同的函数地址来实现不同的自动加载机制。

spl\_autoload是SPL实现的默认的自动加载函数，它的功能比较简单。它可以接收两个参数，第一个参数是$class\_name，表示类名，第 二个参数$file\_extensions是可选的，表示类文件的扩展名，可以在$file\_extensions中指定多个扩展名，护展名之间用分号隔 开即可；如果不指定的话，它将使用默认的扩展名**.inc****或****.php**。spl\_autoload首先将$class\_name变为小写，然后在所有的 include path中搜索$class\_name.inc或$class\_name.php文件(如果不指定$file_extensions参数的话)，如果找 到，就加载该类文件。

在PHP脚本中第一次调用spl\_autoload\_register()时不使用任何参数，就可以将autoload\_func指向spl\_autoload。

通过上面的说明我们知道，spl\_autoload的功能比较简单，而且它是在SPL扩展中实现的，我们无法扩充它的功能。如果想实现自己的更灵活的自动加载机制怎么办呢？这时，spl\_autoload_call函数闪亮登场了。

我们先看一下spl\_autoload\_call的实现有何奇妙之处。在SPL模块内部，有一个全局变量autoload\_functions，它本质上 是一个HashTable，不过我们可以将其简单的看作一个链表，链表中的每一个元素都是一个函数指针,指向一个具有自动加载类功能的函数。 spl\_autoload\_call本身的实现很简单，只是简单的按顺序执行这个链表中每个函数，在每个函数执行完成后都判断一次需要的类是否已经加载， 如果加载成功就直接返回，不再继续执行链表中的其它函数。如果这个链表中所有的函数都执行完成后类还没有加载，spl\_autoload_call就直接 退出，并不向用户报告错误。因此，使用了autoload机制，并不能保证类就一定能正确的自动加载，关键还是要看你的自动加载函数如何实现。

spl\_autoload\_register函数可以将用户定义的自动加载函数注册到这个链表中，并将autoload\_func函数指针指向spl\_autoload\_call函数。也可以通过spl\_autoload\_unregister函数将已经注册的函数从autoload\_functions链表中删除。

现在回到第一节最后的问题，我们有了解决方案：根据每个类库不同的命名机制实现各自的自动加载函数，然后使用spl\_autoload\_register分别将其注册到SPL自动加载函数队列中就可了。这样我们就不用维护一个非常复杂的__autoload函数了。

**4、autoload效率问题及对策**

autoload机制本身并不是影响系统效率的原因，甚至它还有可能提高系统效率，因为它不会将不需要的类加载到系统中。实际上，影响autoload机制效率本身恰恰是用户设计的自动加载函数。如果它不能高效的将类名与实际的磁盘文件(注意，这里指实际的磁盘文件，而不仅 仅是文件名)对应起来，系统将不得不做大量的文件是否存在(需要在每个include path中包含的路径中去寻找)的判断，而判断文件是否存在需要做磁盘I/O操作，众所周知磁盘I/O操作的效率很低，因此这才是使得autoload机 制效率降低的罪魁祸首!

因此，我们在系统设计时，需要定义一套清晰的将类名与实际磁盘文件映射的机制。这个规则越简单越明确，autoload机制的效率就越高。

结论：autoload机制并不是天然的效率低下，只有滥用autoload，设计不好的自动装载函数才会导致其效率的降低。