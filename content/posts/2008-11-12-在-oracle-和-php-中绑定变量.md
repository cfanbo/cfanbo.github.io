---
title: 在 Oracle 和 PHP 中绑定变量
author: admin
type: post
date: 2008-11-12T10:46:49+00:00
excerpt: |
 |
 想必您一定知道，当前的大多数网站都依赖数据库，只是方式各有不同。 无论您正在构建的站点需要论坛、电子商务组件、包含大量文章和信息还是仅仅从访问者那里获得反馈，您都很可能会通过某种方式并入数据库。 尽管数据库很重要并通常是不可或缺的，但使用它们会影响（通常是不利影响） Web 应用程序的两个方面： 性能和安全性。 了解何时以及如何在 PHP 中绑定变量将对改善这两个问题方面大有帮助。


 如果您曾经对 Web 项目进行过测试，想必您一定会知道数据库交互通常是要求最高的过程。 在数据库中运行查询时，Oracle 必须先对查询进行分析，以确保它的语法正确，然后才执行实际的查询。 即使您运行多个相似查询也必须先进行分析：
url: /archives/582
IM_data:
 - 'a:1:{s:53:"http://oracleimg.com/admin/images/ocom/bullet_5x5.gif";s:70:"http://blog.haohtml.com/wp-content/uploads/2009/06/da69_bullet_5x5.gif";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---
作者：Larry Ullman通过绑定变量提高 Oracle 驱动的 PHP 应用程序的速度和安全性。本文相关下载：

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle 数据库 10 _g_](http://www.oracle.com/technology/global/cn/software/products/database/oracle10g/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle Instant Client](http://www.oracle.com/technology/global/cn/software/tech/oci/instantclient/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle JDeveloper PHP Extension](http://www.oracle.com/technology/global/cn/products/jdev/htdocs/partners/addins/exchange/php/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Zend Core for Oracle](http://www.oracle.com/technology/global/cn/tech/php/zendcore/index.html)

想必您一定知道，当前的大多数网站都依赖数据库，只是方式各有不同。 无论您正在构建的站点需要论坛、电子商务组件、包含大量文章和信息还是仅仅从访问者那里获得反馈，您都很可能会通过某种方式并入数据库。 尽管数据库很重要并通常是不可或缺的，但使用它们会影响（通常是不利影响） Web 应用程序的两个方面： 性能和安全性。 了解何时以及如何在 PHP 中绑定变量将对改善这两个问题方面大有帮助。



如果您曾经对 Web 项目进行过测试，想必您一定会知道数据库交互通常是要求最高的过程。 在数据库中运行查询时，Oracle 必须先对查询进行分析，以确保它的语法正确，然后才执行实际的查询。 即使您运行多个相似查询也必须先进行分析：

```
SELECT * FROM movies WHERE movie_id=1
SELECT * FROM movies WHERE movie_id=26
SELECT * FROM movies WHERE movie_id=5689
```

尽管这三个查询之间的唯一差别体现在所获取的精确记录上，但 Oracle 仍将单独处理它们，并在执行之前分别对它们进行分析。 绑定变量的第一个好处是，Oracle 只需分析查询一次，而不管它究竟使用不同的值运行了多少次。 这种在脚本方法方面的改变可以极大地提高性能。



作为 Web 开发人员，您通常遇到的第二个问题是站点的安全性。 由于该问题体现在很多方面，因此找到解决它的方法无异于一场永无休止但却至观重要的战役。 在数据库驱动的站点中，许多查询都依赖于外部值，如用户从表单中提交的值、在 URL 中传递给页面的值等等。 此类查询很容易受到 SQL 注入攻击的破坏。 （“SQL 注入攻击”是指恶意用户在尝试破坏查询的过程中向 PHP 脚本提供无效数据。） 如果对查询的处理方法不当，恶意用户便有可能从生成的错误消息中了解一些有关脚本、数据库或服务器的信息。 以如下所示的查询为例：

```
SELECT * FROM movies WHERE movie_id=$_GET['id']
```

想必您在看到该查询时一定会吃惊不已，因为它的安全性实在是太差了。 用户只需更改 URL（例如，将 http://www.example.com/movie.php?id=23 更改为 http://www.example.com/movie.php?id=HaHa!）便可以破坏此查询。 当然，查询中使用的所有数据都应进行验证，但每当在查询中使用变量时，如果变量的值不同于预期的值，便有可能导致出现错误。 由于绑定变量与实际的查询分离，因此可以大大降低 SQL 注入攻击的可能性。



在这篇“Oracle+PHP 指南”操作文档中，您将了解如何在 PHP 脚本中执行 Oracle 查询时绑定变量。 通过将以下技巧和代码示例应用于您自己的 Web 应用程序，您可以轻松地提高它们的性能和安全性。



背景知识/概述



实际示例对于演示绑定变量的用法再合适不过了。 我所要介绍的是我几年前编写的一个应用程序，通过它，高尔夫专业服务公司可以规定他们的球场提供的开球时段和收费标准。 例如，他们可能规定在某个特定的周六，可利用的开球时段为上午 7 点到下午 4 点，间隔为 10 分钟，下午 2 点之前的收费标准为 50 美元，2 点之后为 40 美元。 这些值源于 HTML 表单；负责处理的 PHP 脚本随后在数据库表中为每个开球时段创建一个记录（这样，高尔夫球手便可以从记录列表中在线选择一个时间）。 仅仅为了表示一天，该进程就可能需要 50 个或更多个极其相似的 `INSERT` 查询，这种情况下使用绑定变量将比较合适。



可以使用以下 SQL 语句创建该示例的简化表结构（没有其他表，我已经删除了标识键等内容）：

```
CREATE TABLE teetimes (
	teetime DATE,
	rate NUMBER(5,2)
)
```

显而易见，可以通过多种方法对该示例进行扩展。 但现在最重要的是，本操作文档中的代码假设您已经建立了这样一个表并可以从 PHP 脚本连接并填充它。



以下步骤将演示实现绑定变量需要执行的确切操作。 最终代码将通过一系列步骤构建，并对每个进程进行分析，以便您了解它的用途。 在 PHP 脚本中并入绑定变量分为如下所示的基本步骤：

1. 建立所使用的查询。

2. 针对绑定变量重写查询。

3. 在 Oracle 中分析基本查询。

4. 在 PHP 中向变量分配值。

5. 执行查询。


第 1 步： 定义查询



清单 1 是将一些记录插入到 teetimes 表中的通用 PHP 脚本概要。 与 Oracle 相关的代码假设您使用 PHP 5，其中的 OCI 函数所采用的命名模式和语法与 PHP 4 中的 OCI 函数略有不同，但一致性更高。如果您使用较旧版本的 PHP，请查看 _PHP 手册_以了解正确的函数和语法（如果需要的话）。 此外，由于 PHP 与 Oracle 之间的通信可能存在很多疑难问题，因此您需要阅读[此疑难解答指南][1]{.bodylink}以了解其他用于处理环境变量的方法。



**清单 1**

```

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
	<meta http-equiv="content-type" content="text/html; charset=iso-8859-15" />
	<title>Binding Variables with PHP</title>

</head>
<body>
<h3>Entering Tee Times</h3>
<?php // bind1.php - listing 1

// Establish the environmental variables.
$sid = 'test10g';
$home = '/Users/oracle/10gEAR2/orahome';

putenv("ORACLE_HOME=$home");
putenv("ORACLE_SID=$sid");
putenv("TNS_ADMIN=$home/network/admin");

// Create the array of data to be inserted.
// This data represents what __should__ come from an HTML form.
$teetimes = array();
$date = '2005-08-20';

// Loop through each available hour in the day.
for ($hour = 7; $hour <! 16; $hour++) {

    // Loop through each hour in 10 minute increments.
    for ($minute = 0; $minute <! 60; $minute += 10) {

        // Create the date and time value.
        $this_time = "$date $hour:$minute";

        // Add a 0 if necessary.
        if ($minute <! 10) $this_time .= '0';

        // Determine the rate to use.
        $rate = ($hour <! 14) ? 50.00 : 40.00;

        // Add this teetime and rate to the array.
        $teetimes[$this_time] = $rate;

    }

}

//echo '<!pre>' . print_r ($teetimes, 1) . '<!/pre>'; // For debugging

// Connect to Oracle.

$c = oci_pconnect ('scott', 'tiger', $sid) OR die
  ('Unable to connect to the database.ERROR: <!pre>' . print_r(oci_error(),1) . '<!/pre><!/body><!/html>');

// Insert each record into the table.
foreach ($teetimes as $time => $rate) {

    // Make the query, for example:
    /* INSERT INTO teetimes (teetime, rate) VALUES (TO_DATE('2005-08-21 15:00', 'yyyy-mm-dd hh24:mi'), 40.00); */
    $q = "INSERT INTO teetimes (teetime, rate) VALUES (TO_DATE('$time', 'yyyy-mm-dd hh24:mi'), $rate)";

    // Run the query.
	$s = oci_parse($c, $q);
     oci_execute ($s);

}

// 关闭连接。

oci_close($c);

// Query to confirm the results:
/* SELECT TO_CHAR(teetime, 'MONTH DD, YYYY HH:MI AM') AS "Tee Time", rate FROM teetimes ORDER BY teetime ASC */
?>
</body>
</html>

```

该脚本的用途很简单： 假设它将从表单中收到一组日期、时间、增量和费用。 应采用相应的语法组装该数据，然后将其插入到数据库中。 由于尚未创建 HTML 表单，因此该脚本将自动生成一组代表性数据。 然后，将每个单独的开球时间插入到 Oracle 的循环中。 尽管该脚本可以高效、正常地运行，但采用绑定变量可以显著提高它的性能。



第 2 步： 使用标识符重新定义查询



在清单 1 中可以看到，该查询最初定义为：

```
INSERT INTO teetimes (teetime, rate) VALUES (TO_DATE('$time', 'yyyy-mm-dd hh24:mi'), $rate)
```

其中， `$time` 和 `$rate` 是从生成的数据数组中提取的。 现在必须转换此查询，以便用占位符代替变量来表示不断变化的数据。 要使用的语法为 `:marker`，其中的 marker 可以是任何标识符。 在这个特殊实例中，应将该查询转换为

```
INSERT INTO teetimes (teetime, rate) VALUES (TO_DATE(:t, 'yyyy-mm-dd hh24:mi'), :r)
```

在下面的清单 2（即最终的结果）中，该查询已经从它在清单 1 中的位置移走。现在，在循环的外部定义它，这是因为只需定义它一次（与为插入的每个记录定义一次相反）。 还要注意的是，在绑定变量时，您甚至可以删除通常情况下所需的引号（例如， `TO_DATE()` 函数中的第一个参数只是 `:t`，而非 `'$time'`。） 这是因为变量实际上与查询语法分离。



**清单 2**

```

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
	<meta http-equiv="content-type" content="text/html; charset=iso-8859-15" />
	<title>Binding Variables with PHP</title>
</head>
<body>
<h3>Entering Tee Times</h3>
<?php

// Establish the environmental variables.
$sid = 'test10g';
$home = '/Users/oracle/10gEAR2/orahome';

putenv("ORACLE_HOME=$home");
putenv("ORACLE_SID=$sid");
putenv("TNS_ADMIN=$home/network/admin");

// Create the array of data to be inserted.
// This data represents what __should__ come from an HTML form.
$teetimes = array();
$date = '2005-08-21';

// Loop through each available hour in the day.
for ($hour = 7; $hour < 16; $hour++) {

    // Loop through each hour in 10 minute increments.
    for ($minute = 0; $minute < 60; $minute += 10) {

        // Create the date and time value.
        $this_time = "$date $hour:$minute";

        // Add a 0 if necessary.
        if ($minute < 10) $this_time .= '0';

        // Determine the rate to use.
        $rate = ($hour < 14) ? 50.00 : 40.00;

        // Add this teetime and rate to the array.
        $teetimes[$this_time] = $rate;

    }

}

//echo '<pre>' . print_r ($teetimes, 1) . '</pre>'; // For debugging

// Connect to Oracle.

$c = oci_pconnect ('scott', 'tiger', $sid) OR die
  ('Unable to connect to the database.ERROR: <pre>' . print_r(oci_error(),1) . '</pre></body></html>');

// Define the query.
$q = "INSERT INTO teetimes (teetime, rate) VALUES (TO_DATE(:t, 'yyyy-mm-dd hh24:mi'), :r)";

// Parse the query.

$s = oci_parse($c, $q);

// Bind the values.

oci_bind_by_name($s, ':t', $time, 16);
oci_bind_by_name($s, ':r', $rate, 5);

// Insert each record into the table.
foreach ($teetimes as $time => $rate) {

    // Execute the query.
    oci_execute ($s);

}

// 关闭连接。

oci_close($c);

// Query to confirm  the results:
/* SELECT TO_CHAR(teetime, 'MONTH DD, YYYY HH:MI AM') AS "Tee Time", rate FROM teetimes ORDER BY teetime ASC */

?>
</body>
</html>

```

第 3 步： 在 Oracle 中分析查询



使用 Oracle 的标识符定义查询后，应通过 Oracle 对它进行分析。 Oracle 对每个查询均进行分析，以确保语法正确。 在 PHP 脚本中，请使用 `oci_parse()` 函数（在 PHP 4 中，请使用 `OCIParse()`），并为该函数提供数据库连接和查询作为它的参数：

```
$s = oci_parse ($c, $q);
```

分析结果仍分配给表示该语句的变量 ( `$s`)，这与您使用非绑定变量处理查询的方法完全相同。



第 4 步： 将 PHP 变量与标识符关联



如果您留意的话，并且如果您属于善于提问的一类人，那么您现在很想知道 PHP 中的值如何作为查询的一部分运行。 您可以使用 `oci_bind_by_name()` 函数（在 PHP 4 中为 `OCIBindByName()`）执行此操作。 该函数将语句资源作为它的第一个参数，将标识符的名称作为它的第二个参数，并将 PHP 变量的名称（也可以是文字值）作为它的第三个参数。例如：

```
oci_bind_by_name($s, ':t, $time);
oci_bind_by_name($s, ':r, $rate);
```

为了确保安全并最大限度地降低出现 Oracle 错误的可能性，最好使用第四个可选参数： 要插入的数据的最大长度。 最终的绑定行为：

```
oci_bind_by_name($s, ':t, $time, 16);
oci_bind_by_name($s, ':r, $rate, 5);
```

以上使用的数字分别对应于 `$time` 和 `$rate` 的最大合理长度。 （或者，如果将 -1 用于第四个参数，PHP 将把变量的当前长度用作最大长度。） 您会再次发现，在该示例脚本（参阅清单 2）中，这两行位于 foreach 循环之前。 这似乎有点另人不解，因为 `$time` 和 `$rate` 此时并没有值。 下面解释了这样做的原因： 在实际运行查询时，这些行指示 Oracle 将 `$time` 中存储的值用于 `:t`，并将 `$rate` 中存储的值用于 `:r`。 只要在执行查询时这两个变量包含值，一切便会正常进行。



第 5 步： 执行绑定查询



最后一步是针对要插入的每一组值执行查询。 在将要访问每个数组元素的循环中，您只需分别将相应的值分配给 `$time` 和 `$rate`，然后即可执行查询。 同一 `oci_execute()` 函数（或 PHP 4 中的 `OCIExecute()`）使用与未绑定版本相同的语法执行以下代码：

```
oci_execute ($s);
```

该操作再次在 foreach 循环中执行，其中的 `$time` 和 `$rate` 包含它们的相应值。 有关最终的代码，请参见完整的清单 2。



我应指出使用该方法的其他几个好处。 为了方便起见，任何尾随的空白符将从插入的值中删除。 更重要的是，您不必使用 `addslashes()` 或 Magic Quotes 对有问题的字符进行转义（实际上，您根本不应使用），这是因为变量值实际上并不是查询的一部分。



结论



在本操作文档中，您了解了如何使用绑定变量轻松地提高数据库驱动的 Web 应用程序的安全性和性能。 实际并入该技术只涉及几个额外的代码行，而不需要任何特殊的 PHP 扩展或库。



但对于绑定变量，必须注意两件事：

1. 只有那些定期运行、在语法上相同但具有不同值（例如本特殊示例）的查询才能获得快速的性能。 对于复杂难懂的一次性查询，性能方面的好处非常小甚至没有。 为了最终决定是否应在特殊的 Web 应用程序中使用绑定变量，请执行一些基准测试来测试总体效果。

2. 安全性的增强并不意味着取代您自己的标准安全措施。 应始终对查询中使用的任何数据进行验证 – 尤其是来自 `$_POST`、 `$_GET` 或 `$_COOKIE` 中的数据。


您还应知道的是，实际上您可以采用两种方法在 PHP 和 Oracle 之间使用绑定变量。 本文介绍的方法称作绑定参数，这意味着查询参数绑定到变量（有人还将它称作准备语句）。 另一个方法是绑定结果，它是另一种从数据库中检索值（在运行 `SELECT` 查询之后执行）的方法。



有关如何提高 Oracle 和 PHP Web 应用程序的性能的详细信息，请查找有关存储过程（另一种自动化进程的方法）。 谈到性能，您还应了解索引并对表使用索引。 最后，出于整齐目的，应考虑调用 `oci_free_statement()` 函数（在 PHP 4 中为 `OCIFreeStatement()`）以释放与 PHP 中的脚本关联的资源。



* * *

Larry Ullman 是 [DMC Insights Inc.][2]{.bodylink}（一家专门研究信息技术的公司）数字媒体技术部门的总监以及 Web 开发人员主管。 Larry 居住在华盛顿特区的郊外，并曾经编写过多部有关 PHP、SQL、Web 开发和其他计算机技术方面的书籍。

 [1]: http://www.oracle.com/technology/global/cn/tech/php/htdocs/php_troubleshooting_faq.html
 [2]: http://www.dmcinsights.com/