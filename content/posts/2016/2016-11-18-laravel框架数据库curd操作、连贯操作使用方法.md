---
title: Laravel框架数据库CURD操作、连贯操作使用方法
author: admin
type: post
date: 2016-11-18T04:31:00+00:00
url: /archives/17178
categories:
 - 程序开发
tags:
 - Laravel
 - php

---
Laravel是一套简洁、优雅的PHP Web开发框架(PHP Web Framework)。它可以让你从面条一样杂乱的代码中解脱出来；它可以帮你构建一个完美的网络APP，而且每行代码都可以简洁、富于表达力。

**一、Selects**

检索表中的所有行

```
$users = DB::table('users')->get();
foreach ($users as $user)
{
var_dump($user->name);
}

```

**从表检索单个行**

```
$user = DB::table('users')->where('name', 'John')->first();
var_dump($user->name);

```

**检索单个列的行**

```
$name = DB::table('users')->where('name', 'John')->pluck('name');

```

**检索一个列值列表**

```
$roles = DB::table('roles')->lists('title');

```

**该方法将返回一个数组标题的作用。你也可以指定一个自定义的键列返回的数组**

```
$roles = DB::table('roles')->lists('title', 'name');

```



**指定一个Select子句**

```
$users = DB::table('users')->select('name', 'email')->get();
$users = DB::table('users')->distinct()->get();
$users = DB::table('users')->select('name as user_name')->get();

```

**Select子句添加到一个现有的查询**

```
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();

```

**where**

```
$users = DB::table('users')->where('votes', '>', 100)->get();

```

**OR**

```
$users = DB::table('users')->where('votes', '>', 100)->orWhere('name', 'John')->get();

```



**Where Between**

```
$users = DB::table('users')->whereBetween('votes', array(1, 100))->get();

```



**Where Not Between**

```
$users = DB::table('users')->whereNotBetween('votes', array(1, 100))->get();

```



**Where In With An Array**

```
$users = DB::table('users')->whereIn('id', array(1, 2, 3))->get();
$users = DB::table('users')->whereNotIn('id', array(1, 2, 3))->get();

```



**Using Where Null To Find Records With Unset Values**

```
$users = DB::table('users')->whereNull('updated_at')->get();

```



**Order By, Group By, And Having**

```
$users = DB::table('users')->orderBy('name', 'desc')->groupBy('count')->having('count', '>', 100)->get();

```

**Offset & Limit**

```
$users = DB::table('users')->skip(10)->take(5)->get();

```

**二、连接**

**Joins**

查询构建器也可以用来编写连接语句。看看下面的例子:

**Basic Join Statement**

```
DB::table('users')
->join('contacts', 'users.id', '=', 'contacts.user_id')
->join('orders', 'users.id', '=', 'orders.user_id')
->select('users.id', 'contacts.phone', 'orders.price')
->get();

```



**左连接语句**

```
DB::table('users')
->leftJoin('posts', 'users.id', '=', 'posts.user_id')
->get();
DB::table('users')
->join('contacts', function($join)
{
$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
})
->get();
DB::table('users')
->join('contacts', function($join)
{
$join->on('users.id', '=', 'contacts.user_id')
->where('contacts.user_id', '>', 5);
})
->get();

```



**三、分组**

有时候,您可能需要创建更高级的where子句,如“存在”或嵌套参数分组。Laravel query builder可以处理这些:

```
DB::table('users')
->where('name', '=', 'John')
->orWhere(function($query)
{
$query->where('votes', '>', 100)
->where('title', '<>', 'Admin');
})
->get();

```



上面的查询将产生以下SQL:

```
select * from users where name = 'John' or (votes > 100 and title
<> 'Admin')
Exists Statements
DB::table('users')
->whereExists(function($query)
{
$query->select(DB::raw(1))
->from('orders')
->whereRaw('orders.user_id = users.id');
})
->get();

```



上面的查询将产生以下SQL:

```
select * from userswhere exists (
select 1 from orders where orders.user_id = users.id
)

```



**四、聚合**

查询构建器还提供了各种聚合方法,如统计,马克斯,min,avg和总和。

Using Aggregate Methods

```
$users = DB::table('users')->count();
$price = DB::table('orders')->max('price');
$price = DB::table('orders')->min('price');
$price = DB::table('orders')->avg('price');
$total = DB::table('users')->sum('votes');

```

**Raw Expressions**

有时您可能需要使用一个原始表达式的查询。这些表达式将注入的查询字符串,所以小心不要创建任何SQL注入点!创建一个原始表达式,可以使用DB:rawmethod:

**Using A Raw Expression**

```
$users = DB::table('users')
->select(DB::raw('count(*) as user_count, status'))
->where('status', '<>', 1)
->groupBy('status')
->get();

```

```
$total = self::whereRaw("id=$id or fid=$from_id")->whereIn('status', [1,2,3])->count();
\DB::listen(function($sql,$binds){
  dd($sql, $binds);//输出sql看看是否正确
});

```

**递增或递减一个列的值**

```
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);

```



**您还可以指定额外的列更新:**

```
DB::table('users')->increment('votes', 1, array('name' => 'John'));

```



Inserts

**将记录插入表**

```
DB::table('users')->insert(
array('email' => 'john@example.com', 'votes' => 0)
);

```



**将记录插入表自动增加的ID**

如果表,有一个自动递增的id字段使用insertGetId插入一个记录和检索id:

```
$id = DB::table('users')->insertGetId(
array('email' => 'john@example.com', 'votes' => 0)
);

```



注意:当使用PostgreSQL insertGetId方法预计,自增列被命名为“id”。

**多个记录插入到表中**

```
DB::table('users')->insert(array(
array('email' => 'taylor@example.com', 'votes' => 0),
array('email' => 'dayle@example.com', 'votes' => 0),
));

```

**四、Updates**

更新一个表中的记录

```
DB::table('users')
->where('id', 1)
->update(array('votes' => 1));

```

**五、　Deletes**

**删除表中的记录**

```
DB::table('users')->where('votes', '<', 100)->delete();

```

**删除表中的所有记录**

```
DB::table('users')->delete();

```

**删除一个表**

```
DB::table('users')->truncate();

```



**六、Unions**

查询构建器还提供了一种快速的方法来“联盟”两个查询:

代码如下:

```
$first = DB::table('users')->whereNull('first_name');
$users =
DB::table('users')->whereNull('last_name')->union($first)->get();

```



unionAll方法也可以,有相同的方法签名。

Pessimistic Locking

查询构建器包括一些“悲观锁定”功能来帮助你做你的SELECT语句。　　运行SELECT语句“共享锁”,你可以使用sharedLock方法查询:

```
$users = DB::table('users')->remember(10)->get();

```

更新“锁”在一个SELECT语句,您可以使用lockForUpdate方法查询:

```
$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();
```


**七、缓存查询**

你可以轻松地缓存查询的结果使用记忆法:

```
$users = DB::table('users')->remember(10)->get();
```

在本例中,查询的结果将为十分钟被缓存。查询结果缓存时,不会对数据库运行,结果将从默认的缓存加载驱动程序指定您的应用程序。　　如果您使用的是支持缓存的司机,还可以添加标签来缓存:

```
$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();

```