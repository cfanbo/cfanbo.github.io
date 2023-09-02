---
title: 在 Oracle 和 PHP 中使用 LOB
author: admin
type: post
date: 2008-11-12T10:50:19+00:00
excerpt: |
 |
 使用 VARCHAR2 这样的 Oracle 类型是完全可以的，但如果您要一次性存储的数据量超过它的 4,000 字节的极限，情况将会如何？ 要完成此任务，您需要 Oracle 的某个 Long 对象 (LOB) 类型，为此您应了解如何使用 PHP API 来处理 LOB。 这对于不熟悉它的人来说是很困难的。


 在这篇“Oracle+PHP 指南”操作文档中，您将了解可用的 LOB 类型以及与它们相关的问题，然后将探讨 PHP 中常见 LOB 操作示例。


 Oracle 中的 Long 对象
url: /archives/585
IM_data:
 - 'a:1:{s:53:"http://oracleimg.com/admin/images/ocom/bullet_5x5.gif";s:70:"http://blog.haohtml.com/wp-content/uploads/2009/06/da69_bullet_5x5.gif";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---
作者：Harry Fuecks是否达到 4,000 字节的极限？ 我们先来了解一下 LOB……本文相关下载：

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Oracle 数据库 10 _g_](http://www.oracle.com/technology/global/cn/software/products/database/oracle10g/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Zend Core for Oracle](http://www.oracle.com/technology/global/cn/tech/php/zendcore/index.html)

![](http://oracleimg.com/admin/images/ocom/bullet_5x5.gif)[Apache HTTP Server 1.3 和更高版本](http://httpd.apache.org/download.cgi)

使用 VARCHAR2 这样的 Oracle 类型是完全可以的，但如果您要一次性存储的数据量超过它的 4,000 字节的极限，情况将会如何？ 要完成此任务，您需要 Oracle 的某个 Long 对象 (LOB) 类型，为此您应了解如何使用 PHP API 来处理 LOB。 这对于不熟悉它的人来说是很困难的。



在这篇“Oracle+PHP 指南”操作文档中，您将了解可用的 LOB 类型以及与它们相关的问题，然后将探讨 PHP 中常见 LOB 操作示例。



Oracle 中的 Long 对象



Oracle 提供了以下 LOB 类型：

- BLOB，用于存储二进制数据

- CLOB，用于使用数据库字符集编码存储字符数据

- NCLOB，用于使用国家字符集存储 Unicode 字符数据。 注意，您将在本文中使用的 PHP [OCI8](http://www.php.net/oci8) 扩展当前不支持 NCLOB。

- BFILE，用于引用存在于操作系统的文件系统中的外部文件


LOB 的更深一层的子类别是临时 LOB（可以为 BLOB、CLOB 或 NCLOB），它在被释放之前一直存储在临时表空间中。



注意，较旧版本的 Oracle 分别为字符和二进制数据提供了 LONG 和 LONG RAW 类型。 在 Oracle9_i_ 中，LOB 取代了这两个类型。



**LOB 存储。** 对于 BLOB、CLOB 和 NCLOB 类型，Oracle 数据库 10_g_ 能够在单个值中最多存储 128TB 数据，具体情况取决于为 LOB 定义的数据库块大小和“块”设置。



LOB 本身由两个元素构成： LOB 内容和 LOB 定位器（它是指向 LOB 内容的“指针”）。 这种划分对于 Oracle 高效地存储和管理 LOB 是必需的，它反映在用于对 LOB 执行 `INSERT`、 `UPDATE` 和 `SELECT` 操作的 PHP API 中（如下所示）。



对于内部 LOB 类型（即 BFILE 以外的类型），如果 LOB 小于 4KB，则 Oracle 将 LOB 的内容“整齐”地存储在表中（与行的剩余部分存储在一起）。 默认情况下，大于 4KB 的 LOB“不规则”地存储在表的表空间中。 该方法可以快速检索到小型 LOB，而对于大型 LOB，访问时间将比较长，但扫描表时的总体性能保持不变。



适用于 LOB 存储和访问并可以提高性能的可选方法还有很多（如内存缓存和缓冲），具体使用哪一个应根据应用程序的具体情况而定。 有关进一步的信息，请参阅 Oracle 文档中的 [LOB 性能指南][1]{.bodylink}和 [Oracle 数据库应用程序开发人员指南 – 大型对象][2]{.bodylink}。



**有关 LOB 使用方面的限制。** LOB 类型在使用方面存在一些限制，最重要的限制体现在它们在 SQL 语句中的使用。 您不能在以下任意查询中使用 LOB 类型。

```
SELECT DISTINCT <lob_type>
ORDER BY <lob_type>
GROUP BY <lob_col>
```

将 LOB 类型列用于表连接、 `UNION`、 `INTERSECTION` 和 `MINUS` 语句也是非法的。



在 LOB 使用的其他方面存在更多限制，例如，您不能将 LOB 用作主键列。 有关详细信息，请再次参阅 [Oracle 数据库应用程序开发人员指南 – 大型对象][2]{.bodylink}。



CLOB 和字符集



数据库的默认字符集由参数 NLS_CHARACTERSET 定义，应使用该字符集对位于 CLOB 中的文本进行编码。 使用以下 SQL 确定数据库字符集编码：

```
SELECT value FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET'
```

鉴于 PHP 中缺乏对 NCLOB 的支持，因此您可能需要考虑将 Unicode 编码（如 UTF-8）用作数据库字符，这可以通过以下语句实现（假设您有足够的权限）：

```
ALTER DATABASE CHARACTER SET UTF8
```

注意： 在您不了解影响的情况下不要尝试该操作，尤其是如果现有数据或应用程序代码使用其他字符集。 有关更多信息，请参阅 [Oracle 全球化支持指南][3]{.bodylink}和[全球化 Oracle PHP 应用程序概述][4]{.bodylink}。



使用 LOB



此处将主要介绍 PHP 的 OCI8 扩展。 还有一点值得注意的是，Oracle 提供了 DBMS_LOB 程序包，其中包含用于在 PL/SQL 中使用 LOB 的并行过程和函数。



PHP OCI8 扩展在全局 PHP 命名空间中注册一个称作“OCI-Lob”的 PHP 类。 当您执行 `SELECT` 语句时（例如，其中的某一列的类型为 LOB），PHP 将把它自动绑定到 OCI-Lob 对象实例。 引用 OCI-Lob 对象后，可以调用类似 [ `load()`][5]{.bodylink} 和 [ `save()`][6]{.bodylink} 这样的方法来访问或修改 LOB 的内容。



可用的 OCI-Lob 方法将取决于 PHP 的版本，PHP5 特别提供了 [ `read()`][7]{.bodylink}、[ `seek()`][8]{.bodylink} 和 [ `append()`][9]{.bodylink} 等方法。 _PHP 手册_对提供可用 OCI-Lob 方法的 PHP 版本号介绍得不够明确，因此，如果您存在疑问，可以使用以下脚本进行确认。

```
<?php
foreach (get_class_methods('OCI-Lob') as $method ) {
    print "OCI-Lob::$method()\n";
}
?>
```

在我的系统上运行 PHP 5.0.5 时，我获得了以下方法列表：

```
OCI-Lob::load()
OCI-Lob::tell()
OCI-Lob::truncate()
OCI-Lob::erase()
OCI-Lob::flush()
OCI-Lob::setbuffering()
OCI-Lob::getbuffering()
OCI-Lob::rewind()
OCI-Lob::read()
OCI-Lob::eof()
OCI-Lob::seek()
OCI-Lob::write()
OCI-Lob::append()
OCI-Lob::size()
OCI-Lob::writetofile()
OCI-Lob::writetemporary()
OCI-Lob::close()
OCI-Lob::save()
OCI-Lob::savefile()
OCI-Lob::free()
```

实际上，PHP 4.x OCI8 扩展只支持读取或写入完整的 LOB，这是 Web 应用程序中最常见的用法。 PHP5 对此进行了扩展，从而可以读取和写入 LOB 的“块”，同时还使用 [ `setBuffering()`][10]{.bodylink} 和 [ `getBuffering()`][11]{.bodylink} 方法支持 LOB 缓冲。 PHP5 还提供了独立函数 [ `oci_lob_is_equal()`][12]{.bodylink} 和 [ `oci_lob_copy()`][13]{.bodylink}。



此处的示例将使用新的 PHP5 OCI 函数名（例如，用 [ `oci_parse`][14]{.bodylink} 代替 [ `OCIParse`][15]{.bodylink}）。 这些示例使用以下序列和表：

```
CREATE SEQUENCE mylobs_id_seq
    NOMINVALUE
    NOMAXVALUE
    NOCYCLE
    CACHE 20
    NOORDER
INCREMENT BY 1;

CREATE TABLE mylobs (
    id NUMBER PRIMARY KEY,
    mylob CLOB
)
```

注意，此处的大多数示例都使用 CLOB，但同一逻辑几乎完全可以应用于 BLOB。



插入 LOB



要使用 `INSERT` 插入一个内部 LOB，首先需要使用相应的 Oracle `EMPTY_BLOB` 或 `EMPTY_CLOB` 函数来初始化 LOB，您无法更新一个包含 NULL 值的 LOB。



初始化后，请将该列绑定到 PHP OCI-Lob 对象，然后通过该对象的 `save()` 方法更新 LOB 内容。



以下脚本提供了一个示例，用于从 `INSERT` 查询中返回 LOB 类型：

```
<?php
// connect to DB etc...

$sql = "INSERT INTO
        mylobs
          (
id,
            mylob
          )
       VALUES
          (
            mylobs_id_seq.NEXTVAL,
            --Initialize as an empty CLOB
            EMPTY_CLOB()
          )
       RETURNING
          --Return the LOB locator
          mylob INTO :mylob_loc";

$stmt = oci_parse($conn, $sql);

// Creates an "empty" OCI-Lob object to bind to the locator
$myLOB = oci_new_descriptor($conn, OCI_D_LOB);

// Bind the returned Oracle LOB locator to the PHP LOB object
oci_bind_by_name($stmt, ":mylob_loc", $myLOB, -1, OCI_B_CLOB);

// Execute the statement using , OCI_DEFAULT - as a transaction
oci_execute($stmt, OCI_DEFAULT)
    or die ("Unable to execute query\n");

// Now save a value to the LOB
if ( !$myLOB->save('INSERT: '.date('H:i:s',time())) ) {

    // On error, rollback the transaction
    oci_rollback($conn);

} else {

    // On success, commit the transaction
    oci_commit($conn);

}

// Free resources
oci_free_statement($stmt);
$myLOB->free();

// disconnect from DB etc.
?>
```

注意该示例如何使用事务，它使用 `OCI_DEFAULT` 常量指示 [ `oci_execute`][16]{.bodylink} 等待 [ `oci_commit`][17]{.bodylink} 或 [ `oci_rollback`][18]{.bodylink}。 这一点很重要，因为我在 `INSERT` 中分两个阶段执行操作 – 首先创建行，然后更新 LOB。



注意，如果我使用了 BLOB 类型，则唯一需要更改（假设有一个 BLOB 列）的就是 [ `oci_bind_by_name`][19]{.bodylink} 调用：

```
oci_bind_by_name($stmt, ":mylob_loc", $myLOB, -1, OCI_B_BLOB);
```

或者，您也可以直接绑定字符串而不必指定 LOB 类型；

```
<?php
// etc.

$sql = "INSERT INTO
          mylobs
          (
id,
            mylob
          )
        VALUES
          (
            mylobs_id_seq.NEXTVAL,
:string
          )
";

$stmt = oci_parse($conn, $sql);

$string = 'INSERT: '.date('H:i:s',time());

oci_bind_by_name($stmt, ':string', $string);

oci_execute($stmt)
    or die ("Unable to execute query\n");

// etc.
?>
```

该方法极大地简化了代码，并且在要写入 LOB 的数据相对较小的情况下比较合适。 相反，如果要将大型文件的内容传送到 LOB 中，则可以遍历该文件的内容，方法是对 PHP LOB 对象调用 [ `write()`][20]{.bodylink} 和 [ `flush()`][21]{.bodylink} 以写入较小的块，而不是在单个实例中将整个文件保存在内存中。



选择一个 LOB



如果 `SELECT` 查询包含一个 LOB 列，PHP 将自动把该列绑定到 OCI-Lob 对象。例如：

```
<?php
// etc.

$sql = "SELECT
          *
FROM
          mylobs
        ORDER BY
Id
";

$stmt = oci_parse($conn, $sql);

oci_execute($stmt)
    or die ("Unable to execute query\n");

while ( $row = oci_fetch_assoc($stmt) ) {
    print "ID: {$row['ID']}, ";

    // Call the load() method to get the contents of the LOB
    print $row['MYLOB']->load()."\n";
}

// etc.
?>
```

可以进一步简化以上代码，方法是使用 `OCI_RETURN_LOBS` 常量（与 `oci_fetch_array()` 结合使用）并指示它用 LOB 对象的值替换这些对象：

```
while ( $row = oci_fetch_array($stmt, OCI_ASSOC+OCI_RETURN_LOBS) ) {
    print "ID: {$row['ID']}, {$row['MYLOB']}\n";
}
```

更新 LOB



要使用 `UPDATE` 更新 LOB，也可以在 SQL 中使用“ `RETURNING`”命令（与上面的 `INSERT` 示例一样），但一个更简单的方法是使用 `SELECT ... FOR UPDATE`：

```
<?php
// etc.

$sql = "SELECT
           mylob
FROM
           mylobs
WHERE
           id = 3
        FOR UPDATE /* locks the row */
";

$stmt = oci_parse($conn, $sql);

// Execute the statement using OCI_DEFAULT (begin a transaction)
oci_execute($stmt, OCI_DEFAULT)
    or die ("Unable to execute query\n");

// Fetch the SELECTed row
if ( FALSE === ($row = oci_fetch_assoc($stmt) ) ) {
    oci_rollback($conn);
    die ("Unable to fetch row\n");
}

// Discard the existing LOB contents
if ( !$row['MYLOB']->truncate() ) {
    oci_rollback($conn);
    die ("Failed to truncate LOB\n");
}

// Now save a value to the LOB
if ( !$row['MYLOB']->save('UPDATE: '.date('H:i:s',time()) ) ) {

    // On error, rollback the transaction
    oci_rollback($conn);

} else {

    // On success, commit the transaction
    oci_commit($conn);

}

// Free resources
oci_free_statement($stmt);
$row['MYLOB']->free();

// etc.
?>
```

与 `INSERT` 一样，我需要使用事务执行 `UPDATE`。 一个重要的额外步骤是调用 [ `truncate()`][22]{.bodylink}。 使用 `save()` 更新 LOB 时，它将替换 LOB 的内容，范围从开头一直到新数据的长度。 这意味着较旧的内容（如果它比新内容长）可能仍保留在 LOB 中。



对于 PHP 4.x（其中未提供 `truncate()`），以下替换解决方案使用 Oracle 的 `EMPTY_CLOB()` 函数删除 LOB 中的任何现有内容，然后将新数据保存到其中。

```
$sql = "UPDATE
           mylobs
SET
            mylob = EMPTY_CLOB()
WHERE
           id = 2403
        RETURNING
            mylob INTO :mylob
";

$stmt = OCIParse($conn, $sql);

$mylob = OCINewDescriptor($conn,OCI_D_LOB);

OCIBindByName($stmt,':mylob',$mylob, -1, OCI_B_CLOB);

// Execute the statement using OCI_DEFAULT (begin a transaction)
OCIExecute($stmt, OCI_DEFAULT)
    or die ("Unable to execute query\n");

if ( !$mylob->save( 'UPDATE: '.date('H:i:s',time()) ) ) {

    OCIRollback($conn);
    die("Unable to update lob\n");

}

OCICommit($conn);
$mylob->free();
OCIFreeStatement($stmt);
```

使用 BFILES



使用 BFILE 类型时， `INSERT` 和 `UPDATE` 将该文件在数据库服务器（可能与 Web 服务器不在同一计算机上）的文件系统中的位置告知 Oracle，而不是传递文件内容。 使用 `SELECT` 语句，您可以通过 Oracle 读取 BFILE 的内容（如果您愿意），也可以调用 `DBMS_LOB` 程序包中的函数和过程来获取有关文件的信息。



BFILE 的主要优点是能够直接从文件系统中访问原始文件，同时仍然可以使用 SQL 定位文件。 例如，这意味着 Web 服务器可以直接提供映射，而我可以跟踪包含 BFILES 的表与“users”表（指示哪些用户上载了文件）之间的关系。



在示例中，我首先需要更新上面使用的表模式；

```
ALTER TABLE mylobs ADD( mybfile BFILE )
```

然后，我需要使用 Oracle 注册一个目录别名（这需要管理权限）并授予对该目录的读取权限：

```
CREATE DIRECTORY IMAGES_DIR AS '/home/harryf/public_html/images'
GRANT READ ON DIRECTORY IMAGES_DIR TO scott
```

我现在可以使用 `INSERT` 插入一些如下所示的 BFILE 名称：

```
<?php
// etc.

// Build an INSERT for the BFILE names
$sql = "INSERT INTO
        mylobs
          (
id,
            mybfile
          )
       VALUES
          (
            mylobs_id_seq.NEXTVAL,
            /*
            Pass the file name using the Oracle directory reference
            I created called IMAGES_DIR
            */
            BFILENAME('IMAGES_DIR',:filename)
          )";

$stmt = oci_parse($conn, $sql);

// Open the directory
$dir = '/home/harryf/public_html/images';
$dh = opendir($dir)
    or die("Unable to open $dir");

// Loop through the contents of the directory
while (false !== ( $entry = readdir($dh) ) ) {

    // Match only files with the extension .jpg, .gif or .png
    if ( is_file($dir.'/'.$entry) && preg_match('/\.(jpg|gif|png)$/',$entry) ) {

        // Bind the filename of the statement
        oci_bind_by_name($stmt, ":filename", $entry);

        // Execute the statement
        if ( oci_execute($stmt) ) {
            print "$entry added\n";
        }
    }

}
```

如果需要，我可以通过 Oracle 读取 BFILE 的内容，方法与我在上面选择 CLOB 时所采用的方法相同。 或者，如果我需要获得文件名，则可以直接从文件系统中访问它们，我可以调用如下所示的 `DBMS_LOB.FILEGETNAME` 过程：

```
<?php
// etc.

$sql = "SELECT
id
FROM
          mylobs
WHERE
          -- Select only BFILES which are not null
          mybfile IS NOT NULL;

$stmt1 = oci_parse($conn, $sql);

oci_execute($stmt1)
    or die ("Unable to execute query\n");

$sql = "DECLARE
          locator BFILE;
          diralias VARCHAR2(30);
          filename VARCHAR2(30);

BEGIN

SELECT
            mybfile INTO locator
FROM
            mylobs
WHERE
            id = :id;

          -- Get the filename from the BFILE
          DBMS_LOB.FILEGETNAME(locator, diralias, filename);

          -- Assign OUT params to bind parameters
          :diralias:=diralias;
          :filename:=filename;

END;";

$stmt2 = oci_parse($conn, $sql);

while ( $row = oci_fetch_assoc ($stmt1) ) {

    oci_bind_by_name($stmt2, ":id", $row['ID']);
    oci_bind_by_name ($stmt2, ":diralias", $diralias,30);
    oci_bind_by_name ($stmt2, ":filename", $filename,30);

    oci_execute($stmt2);
    print "{$row['ID']}: $diralias/$filename\n";

}
// etc.
?>
```

此外，您可以使用 `DBMS_LOB.FILEEXISTS` 函数找出操作系统已经删除但在数据库中仍然引用的文件。



结论



本方法文档向您介绍了 Oracle 数据库 10_g_ 提供的不同类型的 LOB，希望您现在已经了解了它们在将大型数据实体高效存储在数据库中这一方面所起到的作用。 您还学习了如何使用 PHP 的 OCI8 API 处理 LOB，其中涵盖了在使用 Oracle 和 PHP 进行开发时将遇到的常见使用情形。



* * *

Harry Fuecks [[http://www.phppatterns.com][23]{.bodylink}] 于 1999 年接触 PHP，此后作为 PHP 开发人员和撰稿人而名声鹊起。他通过 [Sitepoint][24]{.bodylink} 开发人员网络发布了大量初级和中级 PHP 文章，著有 [_The PHP Anthology_][25]{.bodylink}一书。

 [1]: http://www.oracle.com/technology/products/database/application_development/pdf/lob_performance_guidelines.pdf
 [2]: http://download.oracle.com/docs/cd/B19306_01/appdev.102/b14249/toc.htm
 [3]: http://download.oracle.com/docs/cd/B19306_01/server.102/b14225/toc.htm
 [4]: http://www.oracle.com/technology/tech/php/pdf/globalizing_oracle_php_applications.pdf
 [5]: http://www.php.net/manual/en/function.oci-lob-load.php
 [6]: http://www.php.net/manual/en/function.oci-lob-save.php
 [7]: http://www.php.net/manual/en/function.oci-lob-read.php
 [8]: http://www.php.net/manual/en/function.oci-lob-seek.php
 [9]: http://www.php.net/manual/en/function.oci-lob-append.php
 [10]: http://www.php.net/manual/en/function.ocisetbufferinglob.php
 [11]: http://www.php.net/manual/en/function.ocigetbufferinglob.php
 [12]: http://www.php.net/manual/en/function.oci-lob-is-equal.php
 [13]: http://www.php.net/manual/en/function.oci-lob-copy.php
 [14]: http://www.php.net/oci_parse
 [15]: http://www.php.net/ociparse
 [16]: http://www.php.net/oci_execute
 [17]: http://www.php.net/oci_commit
 [18]: http://www.php.net/oci_rollback
 [19]: http://www.php.net/oci_bind_by_name
 [20]: http://www.php.net/manual/en/function.oci-lob-write.php
 [21]: http://www.php.net/manual/en/function.oci-lob-flush.php
 [22]: http://www.php.net/manual/en/function.oci-lob-truncate.php
 [23]: http://www.phppatterns.com/
 [24]: http://www.sitepoint.com/articlelist/210
 [25]: http://www.amazon.com/exec/obidos/tg/detail/-/0957921853/002-8181843-1453612?v=glance