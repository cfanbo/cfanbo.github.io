---
title: PHP类中static 和self的使用区别
author: admin
type: post
date: 2012-12-08T15:21:56+00:00
url: /archives/13521
categories:
 - 程序开发

---

摘自： [http://php.net/manual/en/language.oop5.late-static-bindings.php](http://php.net/manual/en/language.oop5.late-static-bindings.php)

### Limitations of _self::_

Static references to the current class like _self::_ or ___CLASS___ are resolved using the class in which the function belongs, as in where it was defined:


**Example #1 _self::_ usage**

`<code><?php<br />
class A {<br />
public static function who() {<br />
echo __CLASS__;<br />
}<br />
public static function test() {<br />
self::who();<br />
}<br />
}` class B extends A {

public static function who() {

echo __CLASS__;

}

}

B::test();

?>


The above example will output:


```
A
```

### ===================

class A {

public static function who() {

echo __CLASS__;

}

public static function test() {

self::who();

//        static::who();

}

}

A::test();


class B extends A {

public static function who() {

echo __CLASS__;

}

}

echo B::test();


如果使用关键字self运行结果：   A A


如果使用关键字static运行结果：A B


static：父类访问了子类的静态方法


self: 是类内指针，指向本类，静态方法，属性