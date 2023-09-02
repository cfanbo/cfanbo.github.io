---
title: Eclipse与MyEclipse的区别
author: admin
type: post
date: 2011-04-27T12:46:43+00:00
url: /archives/9415
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - Java

---
Eclipse 是一个IDE（Integrated Developing Environment），而这个IDE是允许安装第三方开发的插件来使自身的功能得到扩展和增强的，而Myeclipse就是其中的一种有名的插件集之一，主要是为J2EE开发；MyEclipse将开发者常用到的一些有用的插件都集合起来，提供一种高级编程环境，可以比较轻松完成struts，Hibernate,Spring布局，编写strtus-config.xml；但它是收费的，没Eclipse   MyEclipse是没用的。lomboz也是类似MyEclipse的插件，不过是免费的，当然功能没有MyEclipse好。

 Eclipse是一个开放源代码,基于Java的可扩张的开发平台，多数人都是将Eclipse作为Java的集成开发环境使用，虽然Eclipse使用Java开发：但Eclipse不仅仅局限于Java开发，还可用于其它语言的开发，如C/C++;Eclipse是一个框架和一组服务，它通过各种插件来构建开发环境，因此只要提供支持C/C++ 插件便能进行相应语言的开发．

Eclipse最早是由IBM开发的，后来ＩＢＭ将Eclipse作为一个开发源代码的项目，献给了开源组织Eclipse.org但仍由IBM的子公司ＯＴＩ（主要从事Eclipse开发的人员）继续Eclipse的开发.

（1）MyEclipse 把所有的插件都配好了，直接可以用，比例写jsp,struts,spring之类的，当然包也相当大， 机子不好的话开发程序比较慢，Eclipse 什么都没有，要开发什么就自己配什么插件而已。

（2）严格的说，MyEclipse 只是 Eclipse 体系中的一种插件，只是由于 MyEclipse 经常和 Eclipse 一起安装使用，所以通常也将安装了MyEclipse 插件后的Eclipse叫做MyEclipse，二者可以单独安装，即先装Eclipse之后，再以插件方式安装MyEclipse。另一种方法则是在同时安装二者，即安装MyEclipse，时已经同时安装了Eclipse(他们已经整合在一起)。

MyEclipse为Eclipse提供了一个大量私有和开源的Java工具的集合，这解决了各种开源工具的不一致和缺点。NitroX是一个繁杂而强大的加速Java Web应用开发的工具，还包含了一个强大且能够编译所有JSP和Struts Web应用的工具AppXRay。这些工具解析Java和XML配置文件.

MyEclipse的实际价值来自包含的发布包中的大量的工具。如CCS/JS/HTML/XML的编辑器，帮助创建EJB和Struts项目的向导并产生项目的所有主要的组件如action/session bean/form等。还包含编辑Hibernate配置文件和执行SQL语句的工具。



myeclipse是eclipse的一个插件，主要是用来做WEB方面的开发，里面有Struts Spring Hibernate等的集成，使用起来很方便，如果要用myeclipse6.5建议您下载最新版本的eclipse.甚至，您可以直接在myeclipse官方去下载捆绑了eclipse的myeclipse版本，就更方便了。
Eclipse： [http://www.eclipse.org](http://www.eclipse.org/)
MyEclipse: [http://www.myeclipseide.com](http://www.myeclipseide.com/)

现在网上有好多6.0以上的Myeclipse与eclipse的合版的，两个在一起，大约250M左右，比较好使，公司开发一般用eclipse，因为Myeclipse是收费的，当然学习一般用Myeclipse破解版的，因为比较好使，像做SSH框架的时候，不用自己手动向里加类库，只要一点导入就自动给你导入进去，因为Myeclipse里面给你自带了这些类库。