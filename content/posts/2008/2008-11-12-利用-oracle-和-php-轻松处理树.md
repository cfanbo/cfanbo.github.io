---
title: 利用 Oracle 和 PHP 轻松处理树
author: admin
type: post
date: 2008-11-12T10:43:54+00:00
excerpt: |
 |
 几乎每一种数据驱动的应用程序都依赖于某种形式的、不同复杂程度的层次数据：产品类别中的产品、文件夹中的消息、部门中的员工。当然，某些时候您将需要显示这些数据来创建一个目录、收件箱或组织架构的图表。利用 Oracle 提供的特定供应商的 SQL 扩展和 PHP 在数组处理方面的出色能力，您可以检索并显示一个树，并且以简洁和易于维护的方式对树进行内在的高度优化。

 因为本文讨论的查询和函数都包含较少的过程，而注重提供更条理清晰，易于理解的代码，因此这篇方法文档在实施的时候以及重构现有代码的时候非常有用。如果您的数据拥有树状的数据形式（目前已经显示或者要取其值），那么本方法文档将会很有价值。使用最新推出的优秀 RDBMS 的用户非常幸运，因为新的特性使得一些棘手的处理层次数据的任务变得更为容易 — 虽然自 Oracle8i 起的版本都拥有基本的底层功能。
url: /archives/580
IM_data:
 - 'a:2:{s:53:"http://oracleimg.com/admin/images/ocom/bullet_5x5.gif";s:70:"http://blog.haohtml.com/wp-content/uploads/2009/06/da69_bullet_5x5.gif";s:68:"http://www.oracle.com/technology/pub/images/bollweg_easytrees_f1.gif";s:80:"http://blog.haohtml.com/wp-content/uploads/2009/06/90dc_bollweg_easytrees_f1.gif";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---
作者：Nick Bollweg利用一流的查询和函数，轻松处理层次数据。本文相关下载：

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[示例代码和清单](http://www.oracle.com/technology/pub/files/bollweg_easytrees_sample.zip)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle 数据库 10 _g_ Express 版](http://www.oracle.com/technology/global/cn/software/products/database/oracle10g/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle 即时客户端](http://www.oracle.com/technology/global/cn/software/tech/oci/instantclient/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[为 PHP 提供的 Oracle JDeveloper 扩展](http://www.oracle.com/technology/global/cn/products/jdev/htdocs/partners/addins/exchange/php/index.html)2005 年 12 月发表

几乎每一种数据驱动的应用程序都依赖于某种形式的、不同复杂程度的层次数据：产品类别中的产品、文件夹中的消息、部门中的员工。当然，某些时候您将需要显示这些数据来创建一个目录、收件箱或组织架构的图表。利用 Oracle 提供的特定供应商的 SQL 扩展和 PHP 在数组处理方面的出色能力，您可以检索并显示一个树，并且以简洁和易于维护的方式对树进行内在的高度优化。

因为本文讨论的查询和函数都包含较少的过程，而注重提供更条理清晰，易于理解的代码，因此这篇方法文档在实施的时候以及重构现有代码的时候非常有用。如果您的数据拥有树状的数据形式（目前已经显示或者要取其值），那么本方法文档将会很有价值。使用最新推出的优秀 RDBMS 的用户非常幸运，因为新的特性使得一些棘手的处理层次数据的任务变得更为容易 — 虽然自 Oracle8_i_ 起的版本都拥有基本的底层功能。

了解数据

基本的问题是大多数用户都想以更有意义的方式使用和显示存储在平表中的数据。此类中最常见的一些数据形式有：

- 分类：王国、语系、阶级或国家、城市、县、州

- 系谱：祖父、父亲、孩子

- 机构：总裁、经理、员工或类别、子类别、项目、子项目


每个标准查询的结果行中的值和位置仅指该行，但层次查询返回的结果中的行在树形结构中的一个位置。为了从非层次结果中获取这种结构信息，必须遍历每一个值，在这个过程中进行检查并构建另一种数据结构。避免这个过程而让 Oracle 做它最擅长的事情可消除开发人员与数据交互的步骤。

在下面的示例中，您将使用以上数据形式中的最后一个 — 企业机构。对于具体的数据，我们可使用在 Oracle 数据库 10_g_ Express 版（入门数据库）提供的相对简单的 HR 数据库片段。



![图 1](http://www.oracle.com/technology/pub/images/bollweg_easytrees_f1.gif) 如果您使用了包含 OCI8 扩展的 PHP 编译版本，那么使用以下查询和方法将无需任何特殊的设置。使用数据库抽象类（例如 [Pear:DB][1]{.bodylink}、[ADOdb][2]{.bodylink} 或 PHP 5.1 的 [PDO][3]{.bodylink}）可以提高开发效率，只需更少的代码就可以实现同样的功能。但本方法文档中介绍的层次方法是一个 SQL 扩展，仅适用于 Oracle 用户（并非不鼓励这种方法）。您的代码将不能移植到其他供应商的 RDMBS，而这是使用抽象类的主要目标之一。既然所有的抽象类都将实现类似的功能，下面的示例将使用基础的 OCI 方法。

`CONNECT BY` 连接

第一个查询使用了 `CONNECT BY`：

```
SELECT ENAME, JOB, EMPNO, MGR
FROM EMP
CONNECT BY MGR = PRIOR EMPNO
START WITH MGR IS NULL
```

这里要注意的重要的元素是：

- `CONNECT BY`：这将告诉查询处理程序，您需要获取层次结构。以下表达式将告诉处理程序它需要查看哪些列才能理解该层次结构。

- `PRIOR`：这个特殊的保留字将指示以下值位于一个在行链中位置较高的行中。在本例中，您将查询一个员工的经理是表中的另一个员工的情景。

- `START WITH`：该子句表示层次结构中的起始位置。该数据的特性决定 `NULL` 表示一个没有上级的员工，因此您将查找经理字段为 `NULL` 的员工。在数据建模和实施期间作其它选择时可能需要更多的考虑。下面讨论了 `NOCYCLE`。


然而，当运行了查询时，结果仍然看起来像一张没有明显顺序的平表：

```
ENAME      JOB            EMPNO        MGR
---------- --------- ---------- ----------
KING       PRESIDENT       7839
JONES      MANAGER         7566       7839
SCOTT      ANALYST         7788       7566
ADAMS      CLERK           7876       7788
FORD       ANALYST         7902       7566
SMITH      CLERK           7369       7902
...
```

再仔细地看一下：在总裁之后的前三名员工中的每一个都直接位于其各自的经理的下面。然后，模式变得更复杂：Ford 是一名经理为 Jones 的分析人员，但他位于 Adams 下面。这是以扁平方式显示树的根本问题，因为在父亲下面只能有一个孩子。为了帮助我们理解这种结构，您需要在使用层次查询时自动提供几个虚列中的第一个。在上述查询中修改选中的列的列表，包含 `LEVEL`，这将生成：

```
ENAME      JOB            EMPNO        MGR      LEVEL
---------- --------- ---------- ---------- ----------
KING       PRESIDENT       7839                     1
JONES      MANAGER         7566       7839          2
SCOTT      ANALYST         7788       7566          3
ADAMS      CLERK           7876       7788          4
FORD       ANALYST         7902       7566          3
SMITH      CLERK           7369       7902          4
...
```

每一行现在都带了一个标志，指示它在树状结构中的深度；Ford 和 Scott 拥有相同的级别，而没有更低级别的员工出现在他们之间。他们是树状结构中的同级项。这是您以最佳的方式处理结果所需的基本信息.您需要执行的一个额外的任务是对结果排序 — 如果您按照 ENAME ( `BY ENAME`) 对您仔细构建的 `CONNECT BY` 查询排序 ( `ORDER`)，那么这种巧妙的树状结构将被破坏。为了解决这个问题，在 `ORDER BY` 中添加了 `SIBLINGS` — 在 Oracle9_i_ 中引进的一个构造，根据您设定的规则，把一个共同父项下面的每一个行集放到一个经过排序的列表中。

```
SELECT ENAME, JOB, EMPNO, MGR, LEVEL
FROM EMP
CONNECT BY MGR = PRIOR EMPNO
START WITH MGR IS NULL
ORDER SIBLINGS BY ENAME;
```

查询已经准备就绪，让我们来获取数据。在获取结果之前，您需要与数据库连接，用上面的查询创建一条语句，并让 Oracle 来解析它。（关于完整的程序清单，请查看[示例代码][4]{.bodylink}。）在准备好语句之后，您就可以获取结果了：

```
$nrows = oci_fetch_all($stmt, $results, 0, 0,
				OCI_FETCHSTATEMENT_BY_ROW);
```

注意您必须使用 `OCI_FETCHSTATEMENT_BY_ROW` 标记来将数据放到基于行的表示中而不是默认的基于列的表示中。这对于以易于管理的结构提供待编写的代码非常关键。



处理数据



接下来，您必须编写这种模式中最费力的部分。 `array_map` 使您能够用非常少的句法开销以高度优化的方式将一个用户回调函数应用到数组的每一个元素上。这实现了编程人员时间和处理程序时间节省之间的平衡，使您能够简洁地编写更快速的代码。本示例编造了一个机构项目列表，显示了每一位员工及其工作。这种列表易于显示，此外，它们很好地简化了 CSS，符合 CSS 的风格。

```
function treeFunc( $current ){
	// the previous row's level, or null on the first row
	global $last;

	// structural elements
	$openItem =	'<li>';
	$closeItem =	'</li>';
	$openChildren =	'<ul>';
	$closeChildren =	</ul>
	$structure = "";

	if( !isset( $current['LEVEL'] ) ){
		// add closing structure(s) equal to the very last
		// row's level; this will only fire for the "dummy"
		return str_repeat($closeItem.$closeChildren,
			$last);
	}

	// add the item itself
	$item = "{$current['ENAME']} <i>{$current['JOB']}</i>";

	if ( is_null( $last ) ) {
		// add the opening structure in the case of
		// the first row
		$structure .= $openChildren;
	} elseif ( $last < $current['LEVEL'] ) {
		// add the structure to start new branches
		$structure .= $openChildren;
	} elseif ( $last > $current['LEVEL'] ){
		// add the structure to close branches equal to the
		// difference between the previous and current
		// levels
		$structure .= $closeItem.
			   str_repeat( $closeChildren.$closeItem,
				$last - $current['LEVEL'] );
	} else {
		$structure .= $closeItem;
	}

	// add the item structure
	$structure .= $openItem;

	// update $last so the next row knows whether this row is
	// really its parent
	$last = $current['LEVEL'];

	return $structure.$item;
}
```

上述代码中的大部分反映了行的级别；这与该行上面的行的级别相结合，一次性告诉了您您需要知道的关于该数据的所有信息。如果当前级别高于前面的级别，那么树需要增高。 如果当前级别更低或相同，那么树需要变宽 — 虽然在哪个级别依赖于情况是前者还是后者。理解本部分内容的一个问题是，因为每一行只知道前一行的级别，因此所有未尾的格式化都必须由随后的行来完成。在最后一行的情况下，必须插入一个“虚拟”行来清理格式化。

让 `array_map` 完成工作



现在您获得了一组结果和一个知道如何处理它的函数。您在此编写的 PHP 的目标是使您能够以尽可能方便的方式将这两者结合起来：

```
// you need this value accessible inside the formatting
// function; in an object-oriented approach, this can
// be a class variable
global $last;

// set a value not possible in a LEVEL column to allow the
// first row to know it's "firstness"
$last = null;

// add a dummy "row" to cap off formatting
$results[] = array();

// invoke our formatting function via callback
$formatted = array_map("treeFunc", $results);array_map( "treeFunc", $results );

//output the results
echo implode("\n", $formatted);implode( "\n", $formatted );
```

上述代码使您能够将数据库和结果集的交互操作与个别行的格式化操作隔离开，使代码保持易读和易于维护。它的结果是：

```
• KING PRESIDENT
• BLAKE MANAGER
• ALLEN SALESMAN
• JAMES CLERK
• MARTIN SALESMAN
• TURNER SALESMAN
• WARD SALESMAN
• CLARK MANAGER
• MILLER CLERK
• JONES MANAGER
• FORD ANALYST
• SMITH CLERK
• SCOTT ANALYST
• ADAMS CLERK
```

Oracle9 _i_ 和 Oracle 10 _g_ 的特殊特性

Oracle9_i_ 可完成一些以前需要构建数据结构并迭代遍历它的任务。 `SYS_CONNECT_BY_PATH` 函数将获取一个列名，并将该列名附加到所有子项的值上（加上一个分隔符）。利用它，您可以根据行在层次结构中的位置来构建唯一的 ID，并能够展开分隔符来确定某个 n 代父项或者实施许多其他的技巧。

如果您想知道某行的第一个父项，Oracle 10_g_ 为您提供了一种更为简单的方法： `CONNECT_BY_ROOT`。在任意级别上，这个关键字都将返回特定列之前的第 1 级父项的值，这与上面的 `PRIOR` 非常类似。此外，Oracle 10_g_ 推出了一些构造，以消除为避免常见的层次结构问题而采取的一些变通方法和大量检查工作。典型而言，如果查询处理程序发现它多次检查同一行以确定其父项，那么它将抛出错误。一些数据结构不可避免地包含这种循环，数据所有者认为它们是有益的。在默认这一情况的前提下，在 `CONNECT BY` 后添加 `NOCYCLE` 将允许返回结果，此外还允许填充虚列 CONNECT\_BY\_ISCYCLE，进行分析或显示。



CONNECT\_BY\_ISLEAF 与 CONNECT\_BY\_ISCYCLE 类似，是一个虚列，它提供特定行是否有子行的信息。这可以在过程中稍后为基于 CSS 的格式化提供特别有用的分支。

下一步做什么？

利用页面上的 HTML，仍然可以进行一些有趣且重要的选择。树是否需要变为活动的，允许最终用户展开和合并分支？您是否想以更具空间性的方式进行输出？是否将对包含不同字段值的行进行不同的格式化？这些选择全部取决于您的数据和输出目标，可以通过修改 `treeMethod` 中的不同的结构元素来快速地实现它们。

```
// construct an array acceptable as a callback type
$cbm = new ConnectByMapper();
$formatted = array_map(array($cbm,"treeMethod"), $results);array_map( array( $cbm,"treeMethod" ), $results );
```

在示例代码中还包含了解决该问题的一种面向对象的方法，这为进一步的实验提供了一个基础。

结论



虽然没有在 SQL 的所有实现中提供，但层次查询方法拥有足够的好处，值得鼓励明智的开发人员来了解和使用它们，特别是如果数据包含了多层的层次结构。PHP 特别适用于 SQL 查询的输出，它支持以面向功能和面向对象的方式来处理数据，可供您或下一个维护者方便地对其代码进行增强。请在您的下一次开发过程中考虑这些好处，或者针对现有代码采取相应的方法，以获得这些好处！

 [1]: http://pear.php.net/
 [2]: http://adodb.sourceforge.net/
 [3]: http://us2.php.net/pdo
 [4]: http://www.oracle.com/technology/pub/files/bollweg_easytrees_sample.zip