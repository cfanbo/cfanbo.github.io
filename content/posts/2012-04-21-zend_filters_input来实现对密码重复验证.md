---
title: Zend_Filters_Input来实现对密码重复验证
author: admin
type: post
date: 2012-04-21T09:54:21+00:00
url: /archives/12791
IM_contentdowned:
 - 1
categories:
 - 程序开发

---

### 22.5.4.  使用 Metacommands 来控制过滤器或校验器规则

除了声明从字段到过滤器或校验器的映射，你可以在数组声明中指定一些 “metacommands” ，开控制一些 Zend\_Filter\_Input 的可选的行为。 Metacommands 在给定的过滤器或校验器数组值里以字符串索引条目的形式出现。

#### 22.5.4.1. The `FIELDS` metacommand

如果过滤器或校验器的规则名和需要应用规则的字段名不同，可以用 ‘fields’ metacommand 来指定字段名。


可以用类常量 `Zend_Filter_Input::FIELDS` 而不是字符串来指定这个 metacommand。

```
<?php
$filters = array(
    'month' => array(
        'Digits',        // filter name at integer index [0]
        'fields' => 'mo' // field name at string index ['fields']
    )
);
```

在上例中，过滤器规则使用 ‘digits’ 过滤器给名为 ‘mo’ 的输入字段。 字符串 ‘month’ 变成这个过滤规则的助记键，如果用 ‘fields’ metacommand 指定字段，它不能用做字段名，但可用作规则名。


‘fields’ metacommand 的缺省值是当前规则的索引。在上例中，如果 ‘fields’ metacommand 没有被指定，规则就应用于名为 ‘month’ 的输入字段。


‘fields’ metacommand 的另一个使用是为过滤器或校验器指定字段，这里过滤器或校验器要求多个字段作为输入。 如果 ‘fields’ metacommand 是个数组，过滤器或校验器相应的参数是一个那些字段值的数组。 例如，通常用户会在两个字段中指定密码字符串，他们必需在两个字段中输入相同的字符串。 假定你实现一个校验器类，带有一个数组参数，如果数组中所有的值彼此相等，就返回 `true`。


```
<?php
$validators = array(
    'password' => array(
        'StringEquals',
        'fields' => array('password1', 'password2')
    )
);
// Invokes hypothetical class Zend_Validate_StringEquals, passing an array argument
// containing the values of the two input data fields named 'password1' and 'password2'.
```

如果这个规则校验失败，规则键（ `'password'`）用于 `getInvalid()` 的返回值，不是命名在 ‘fields’ metacommand 中的其它字段。