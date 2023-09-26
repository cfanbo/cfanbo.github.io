---
title: 用 Goalng 开发 OPA 策略
author: admin
type: post
date: 2022-03-31T08:48:03+00:00
url: /archives/31514
toc: true
categories:
 - 程序开发
tags:
 - OPA

---
`Open Policy Agent` 简称`OPA`是一个开源的通用策略引擎，可在整个堆栈中实现统一的、上下文感知的策略实施。OPA 已经成为了云原生计算基金会 ( [CNCF](https://www.cncf.io/)) 领域的毕业项目，已经在 [Kubernetes](https://www.openpolicyagent.org/docs/kubernetes-admission-control.html) / [Istio](https://istio.io) 等多个知名项目里使用 。

OPA的核心思想就是策略即代码。

它使用`Rego`语言开发，Rego 的灵感来自 [Datalog][1]，它是一种易于理解、已有数十年的历史的查询语言。Rego 扩展了 Datalog 以支持 JSON 等文档模型。对于它的详细介绍请参考官方文档 ，这里不再介绍，本方主要介绍如何使用Golang 来开发一个opa策略。

# 概述 

OPA 将 `策略决策` 与 `策略执行` 分离，当您的软件需要做出策略决策时，它会查询 OPA 并提供结构化数据（例如 JSON）作为输入。 OPA 接受任意结构化数据作为输入。![](https://d33wubrfki0l68.cloudfront.net/b394f524e15a67457b85fdfeed02ff3f2764eb9e/6ac2b/docs/latest/images/opa-service.svg)

对于它的输入一般称为 `input`， 可以为任意类型，输出也一样可以为任意类型，即可以输出布尔值 `true` 或 `false`，也可以输出一个 `JSON` 字符串对象。

# 示例 

我们先从官方提供的一个 playground  开始，它是一个官方提供的在线执行平台。

从界面我们可以看出，窗口主要分四块：

 * 左侧是 `Policy`窗口，即上方图的黄色部分
 * 右上方的为用户提供数据的`input` 窗口，也可以直接省略不写，大部分情况下为 `JSON` 或 `YAML` 数据格式
 * 右中间窗口为 `Data` ，提供数据源部分，一般是一个对接持久化数据库的存储层接口。有时规则里有一些数据需要从数据库里查询提取，如根据 `input.user_id` 查询出来用户的入职时间，然后在规则里根据这个入职时间进行一些逻辑判断之类。
 * 右下方则为评估结果输出区域，在 playground 这种方式中会将全部信息都输出给我们，没有办法指定查询指定的字段值，官方提示的示例中输出为布尔值。

下面我们重点只介绍下 `Policy`，它才是我们本节关注的重点。

# Policy 

Rego 策略是使用一组相对较小的类型定义的：`Module`、`Package`和`Imports`、`Rule`、`Expr`和`Term`。从本质上讲，`策略`由`Rule` 组成，这些 `Rule` 由一个或多个`Expr`对策略引擎可用的`文档`进行定义`Expr`由`内在值（Term）`定义，例如`字符串`、`对象`、`变量`等。源码在 。

从大体中来说，主要为 `Package`、`Imports` 和 `Rules` 三部分，而对于 `Rules` 是由多个 `Rule` 组成。

`Rego` 策略通常在文本文件中定义，然后在运行时由策略引擎解析和编译。

解析阶段获取策略的文本或字符串表示，并将其转换为由上述类型组成的`抽象语法树 (AST)`。 AST 的组织如下：

```
Module
 |
 +--- Package (Reference)
 |
 +--- Imports
 |     |
 |     +--- Import (Term)
 |
 +--- Rules
       |
       +--- Rule
             |
             +--- Head
             |     |
             |     +--- Name (Variable)
             |     |
             |     +--- Key (Term)
             |     |
             |     +--- Value (Term)
             |
             +--- Body
                   |
                   +--- Expression (Term | Terms | Variable Declaration)
```

在查询时，策略引擎期望策略已经编译。编译阶段采用一个或多个模块并将它们编译成策略引擎支持的格式。

知道了`AST` 的组织结构，我们再看看官方提供的[示例][2]

```
# 当前所在包声明
package application.authz

# 如果引用了外部包，则需要声明。下面的in关键字后期版本会作为默认关键字，到时候就不再需要导入
# 下面这行是为了介绍包，故意添加上的，官方示例没有这一行
# import future.keywords.in

# 要实现公公宠物主人可以更新宠物的信息, 主人关系是由OPA的输入input(右上方窗口内容)来提供的
# Only owner can update the pet's information
# Ownership information is provided as part of OPA's input

# 声明结果默认值为 false, 如果不写这行的话，当没有规则不满足的时候，不会输出任何内容。可以试着注释掉这行运行看下结果
default allow = false

# 规则内容
allow {
 input.method == "PUT" # 请求类型，常见的有 GET/POST
 some petid # 声明一个值some局部变量
 input.path = ["pets", petid] # 请求路径 /pets/pet113-987
 input.user == input.owner  # 宠物主人
}
```

一个完整的规则 `Rule` 是由 `Head` 和 `Body` 两部分组成。

上面的 `allow` 表示规则变量名，它的完整格式为 `allow := true`，一般推荐使用简写方式，它是组成 `Rule.Head` 部分

`allow{}` 里面的内容是由多个表达式组成, 它是 `Body` 部分。

`allow` 里面的内容每一行称为一个表达式 `Expr` 。 如果在 rego 文件里写规则的话，也可以将多个表达式放在一行，中间分 `;` 分开。

这里的 `input` 是一个 `系统保留字`， 表示用户的输入，它表示下方图的右上角窗口里内容；

第一行的意思是判断右上角窗口里的 `method` 字段的值是否为 `PUT`。

其中第二行使用了[some][3] 关键字声明了一个变量，它也是一个表达式。

整个规则表示当用户通过`PUT` 请求路径 `/pets/xxxxxxx` 时，如果当前的访问用户 `input.user` 是(`==`) 宠物的主人 `input.owner` ，即对象里每个规则结果都`true`，则将 `allow` 赋值为 `true`。

这里规则里有两个判等语句，一个是 `input.method == "PUT"`，另一个是 `input.user == input.owner`。如果在判断的时候遇到第一个为 `false` 的，则终止后续的语句，否则将一直执行直到结束。

为了方便查询，建议先选中窗口的”`Coverage`“选择项，红色的表示有不执行的语句，绿色的表示执行过的语句。如果我们现在点击 “Evalute”按钮的话，可以看到输出结果为

```
{
 "allow": false
}
```

同时在输出结果的上面还显示了本次执行得出的结果数量和消耗的时间。

这是因为 `input.user == input.owner` 这个条件不成立，如果我们试着修改访问用户 `{"user": "bob@hooli.com"}` , 则输出结果为 `true`。![](https://blogstatic.haohtml.com/uploads/2022/05/8175d3cb70c15a34f6d4fd1d622167ed.png)

# Go 开发 

上面的规则如果用Golang 来实现的话，我们应该怎么写呢，在此我们还得要了解一下最上面提供的策略组成层级关系。

## 创建策略规则 

一般我们都是先创建 `RuleSet`，然后再创建多个 Rule 并将其添加到 `RuleSet` 中。

```
// RuleSet
 rs := ast.NewRuleSet()
```

一个 `Rule` 由 `Head` 和 `Body` 和其它字段组成

```
type Rule struct {
    Location *Location `json:"-"`
    Default  bool      `json:"default,omitempty"`
    Head     *Head     `json:"head"`
    Body     Body      `json:"body"`
    Else     *Rule     `json:"else,omitempty"`

    Module *Module `json:"-"`
}
```

其中 Head 指规则名称相关信息，即示例中的 `allow` 字段，而 `Body` 字段则是里面的规则，它是由多个 `表达式` 组成

```
type Head struct {
    Location *Location `json:"-"`
    Name     Var       `json:"name"`
    Args     Args      `json:"args,omitempty"`
    Key      *Term     `json:"key,omitempty"`
    Value    *Term     `json:"value,omitempty"`
    Assign   bool      `json:"assign,omitempty"`
}

type Body []*Expr

// Expr represents a single expression contained inside the body of a rule.
type Expr struct {
    With      []*With     `json:"with,omitempty"`
    Terms     interface{} `json:"terms"`
    Location  *Location   `json:"-"`
    Index     int         `json:"index"`
    Generated bool        `json:"generated,omitempty"`
    Negated   bool        `json:"negated,omitempty"`
}
```

对于表达式，我们直接调用提供的函数即可创建，不同类型的表达式都有自己对应的函数。

我们先创建一个规则集 `RuleSet`，然后动态添加 `Rule` 规则

```
// RuleSet
 rs := ast.NewRuleSet()

 // 添加第一个规则，规则的生成通过从字符串里解析实现
 rs.Add(ast.MustParseRule(`default allow = false`))
```

### 第一个规则 

这里通过 `ast.NewRuleSet()` 函数创建一个规则集合，然后通过 `rs.Add()` 函数添加规则。

规则创建分式有两种：一种是从字符串里直接解析，如上面用到的 ast.NewRuleSet() 函数；另一种是通过提供了函数动态创建，下面会介绍到。我们这里使用了第一种。有些规则也只能通过解析字符串来创建，例如下面 `some v` 这类。

### 第二个规则 

下面我们开始创建第二个规则 `allow`（第 6 – 11 行），我们上面介绍过了 Rule 的结构体，常用的就三个字段 `Head`，`Body` 和 `Else` ，其中 `Else` 字段主要用在 `if-else` 类规则，这里我们没有用到。

#### 第一个表达式 

我们先创建一个 `rule.Body`, 并实现添加第一个规则表达式 `input.method == "PUT"`

```
body := ast.Body{}
 body.Append(
   ast.Equal.Expr(
     ast.RefTerm(ast.VarTerm("input"), ast.StringTerm("method")),
     ast.StringTerm("PUT"),
   ),
 )
```

一个表达式由三部分组成，分别为 `运算符`如`+-*/,=,===` 此类的；还有运算符左侧的元素，一般为变量或字段值；其次就是运算符右侧的元素。

这里实现了判等操作，左侧元素是一个多层级的对象字段，这种情况下需要通过 `ast.RefTerm()` 函数将多个对象连接起来实现。对于只有一个值的来说，直接调用相应类型的函数即可，如字符串类型为 `ast.StringTerm()`, 布尔类型的为 `ast.BooleanTerm()`，类似的还有一些，都是一些常用的其它类型的处理函数。

我们这里的 `input` 是一个变量，所以需要调用 `ast.VarTerm()` 函数来实现，第二个 `method` 是一个字符串，这两个属于同一个Term, 因此这里调用 `ast.RefTerm()` 函数实现。

#### 第二个规则表达式 

表达式 `some petid`

```
body.Append(ast.MustParseExpr("some petid"))
```

对于这类我们只能通过字符串解析的方式实现，没有办法通过调用函数实现。不过此方法也是最简单的方式，避免了使用函数动态创建表达式出错。

#### 第三个表达式 

这个表达式 `input.path = [“pets”， petid]` 稍微有一点复杂，主要是右侧是一个数组对象。

```
body.Append(ast.Equality.Expr(
        ast.RefTerm(
            ast.VarTerm("input"),
            ast.StringTerm("path"),
        ),
        ast.ArrayTerm(
            ast.StringTerm("pets"),
            ast.VarTerm("petid"),
        ),
    ))
```

运算符为赋值运算符，直接调用 `ast.Equality.Expr` 函数创建即可，另外还有一个 `ast.Assign.Expr` 函数用来创建 `:=` 局部变量。左侧部分还是和上面的一样，对于数组一共有两个元素，分别为字符串和变量，分别通过调用函数 `ast.StringTerm()` 和 `ast.VarTerm()` 来实现。我们只要先创建一个数组，再创建两个数组元素即可。

#### 第四个表达式 

```
body.Append(
        ast.Equal.Expr(
            ast.RefTerm(ast.VarTerm("input"), ast.StringTerm("user")),
            ast.RefTerm(ast.VarTerm("input"), ast.StringTerm("owner")),
        ),
    )
```

这个也很简单，基本和第一个表达式一样。

现在第二个规则的四个表达式创建完成，现在我们将其添加到 RuleSet 中即可

```
// Rule
    rs := ast.NewRuleSet()

    r1 := &ast.Rule{
        Head: &ast.Head{
            Name: ast.Var("allow"),
        },
        Body: body,
    }
    rs.Add(r1)
```

这里我们先使用 `&ast.Rule` 创建一个规则，其中 `Head` 是规则名称，也就是 `allow`, 它是可以指定一些参数的，详细见上面 `Head` 的数据结构；而 `Body` 就是我们上面的 `body` 变量。需要注意的是这里的规则一定要使用 `&` 取址即 `&ast.Rule`,千万不要搞错了。

### 生成Module 

创建完所有规则后，我们再看一下 Module

Module 的数据结构为

```
type Module struct {
        Package     *Package       `json:"package"`
        Imports     []*Import      `json:"imports,omitempty"`
        Annotations []*Annotations `json:"annotations,omitempty"`
        Rules       []*Rule        `json:"rules,omitempty"`
        Comments    []*Comment     `json:"comments,omitempty"`
    }
```

`Package` 字段表示当前规则所属包
`Imports` 表示导入了三方的库，当前没有用到
`Rules` 规则，也就是我们上面创建的 `rs := ast.NewRuleSet()`

最后合并代码

```
// Module
    mod := ast.Module{
        Package: &ast.Package{
            Path: ast.Ref{
                ast.StringTerm("policy.rego"),
                ast.StringTerm("application"),
                ast.StringTerm("authz"),
            },
        },
        Rules: rs,
    }
```

这里的 `Package.Path` 表示规则所在的命名空间，创建时 `ast.Ref` 对象的第一个参数可以为任意值，一般填写 policy.rego。从第二个参数开始，后面的元素按先后顺序每个元素就是一个包名，这里规则所属的包名为 `application.authz`。

### 执行并验证 

最后我们验证下它的生成是否正确

```
# format.Ast() 参数是一个地址，一定要小心
    bs, err := format.Ast(&mod)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(string(bs))
```

这里调用 `format.Ast()` 函数输出规则最后生成的 rego 代码，这里一定要注意参数是一个地址值不是普通的值。

## 输入 

上面我们使用 Go 动态创建了规则，规则里用到的一些数据是通过 `input` 提供的。

只有在进行评估的时候才需要提供这些数据。如评估当前登录的用户是否有访问指定资源的权限，这时只有先提供了当前登录用户的信息，然后才能根据规则进行评估，这里的用户登录信息就是我们所说的输入，即 `input`。

下面我们再次使用 Go 来实现数据的输入。

```
// input
	raw := `{
		"method": "PUT",
		"owner": "bob@hooli.com",
		"path": [
			"pets",
			"pet113-987"
		],
		"user": "bob@hooli.com"
	}`
	d := json.NewDecoder(bytes.NewBufferString(raw))

	// Numeric values must be represented using json.Number.
	d.UseNumber()

	// 将字符串转成 go 类的接口
	var input interface{}
	if err := d.Decode(&input); err != nil {
		panic(err)
	}

    r := rego.New(
        rego.Input(input),
        rego.Module("example.rego", string(bs)),
        rego.Query("data.application.authz.allow"),
    )
```

创建一个 rego 对象，指定 input、 规则 和 查询评估结果。

`rego.Input()` 表示输入input，其为 Go 数据类型，它即可以是支持 json 解析的用户自定义 struct

```
type UserInput struct {
	Method string   `json:"method"`
	Owner  string   `json:"owner"`
	Path   []string `json:"path"`
	User   string   `json:"user"`
}
```

也可以直接使用 map

```
module := `
package example.authz

import future.keywords.if
import future.keywords.in

default allow := false

allow if {
    input.method == "GET"
    input.path == ["salary", input.subject.user]
}

allow if is_admin

is_admin if "admin" in input.subject.groups
`

query, err := rego.New(
    rego.Query("x = data.application.authz.allow"),
    rego.Module("example.rego", module),
    ).PrepareForEval(ctx)

if err != nil {
    // Handle error.
}

input := map[string]interface{}{
    "method": "GET",
    "path": []interface{}{"salary", "bob"},
    "subject": map[string]interface{}{
        "user": "bob",
        "groups": []interface{}{"sales", "marketing"},
    },
}

ctx := context.TODO()
results, err := query.Eval(ctx, rego.EvalInput(input))
```

对于 input 即可以在 rego.New() 时指定，也可以在 r.Eval() 时指定。

`rego.Module` 表示当前的评估规则。第一个参数表示当前规则所在的文件名，这里由于是动态创建指定的，所以可以任意填写；第二个参数表示策略规则内容字符串，就是上面的变量 `bs` 内容。两种用法都是支持的。

`rego.Query` 表示要查询的结果。对于评估结果的获取，一般以 `data` 为前缀，如这里 rego.Query(“data.application.authz.allow”) 表示要获取 `application.authz` 包中 `allow` 的评估结果。

## 执行评估 

```
r := rego.New(
        rego.Input(input),
        rego.Module("example.rego", string(bs)),
        rego.Query("data.application.authz.allow"),
    )

  # 评估
    ctx := context.Background()
    q, err := r.PrepareForEval(ctx)
    if err != nil {
        log.Fatal(err)
    }
    resultSet, err1 := q.Eval(ctx)
    if err1 != nil {
        log.Fatal(err)
    }

    fmt.Println("len:", len(resultSet))
    if len(resultSet) > 0 {
        fmt.Printf("text=%stvalue=%#vn", resultSet[0].Expressions[0].Text, resultSet[0].Expressions[0].Value)
    }
    fmt.Println(resultSet.Allowed())
    fmt.Printf("%#v", resultSet)
```

最后输出结果

```
len: 1
text=data.application.authz.allow       value=false
false
rego.ResultSet{rego.Result{Expressions:[]*rego.ExpressionValue{(*rego.ExpressionValue)(0xc00021e300)}, Bindings:rego.Vars{}}}%
```

可以看到，评估结果可以直接调用 `resultSet.Allowed()` 来实现，返回一个布尔值，这里为 `false` 表示无权限。

也可以将上面输入内容 `alice@hooli.com` 修改为 `bob@hooli.com`, 再次执行就可以返回 `true` 表示有权限。

在我们的示例中，只有一个 allow 规则项，一般情况下会有多个，这个时候就可以查询的时候，不需要指定特定的规则名称，如

```
rego.Query("data.application.authz")
```

这样就可以查询出来包中的所有可输出的评估结果。

对于评估结果，上面为了演示方便，部分情况未做考虑。请参考官方建议的处理逻辑 [https://www.openpolicyagent.org/docs/latest/integration/#integrating-with-the-go-api](https://www.openpolicyagent.org/docs/latest/integration/#integrating-with-the-go-api)

```
if err != nil {
    // Handle evaluation error.
} else if len(results) == 0 {
    // Handle undefined result.
} else if result, ok := results[0].Bindings["x"].(bool); !ok {
    // Handle unexpected result type.
} else {
    // Handle result/decision.
    // fmt.Printf("%+v", results) => [{Expressions:[true] Bindings:map[x:true]}]
}
```

开发 `OPA` 时有一点要注意，如果没有为规则名称设置默认值的话，当规则评估结果为 `false` 的话，它的值是不会在输出结果中显示的，因此建议最后在规则前面声明初始化声明一个默认值。

以上完整的代码见： [https://go.dev/play/p/1yMboMz__An](https://go.dev/play/p/1yMboMz__An)

## 常见问题 

对于规则的编写一定要注意对象与数组写法的区别。

例如对象规则

```
input.method == "PUT"
```

正确的写法为

```
ast.Equal.Expr(
    ast.RefTerm(ast.VarTerm("input"), ast.StringTerm("method")),
    ast.StringTerm("PUT"),
),
```

`method` 是对象 `input` 的一个 `字符串` 属性

如果写成

```
ast.Equal.Expr(
    ast.RefTerm(ast.VarTerm("input"), ast.VarTerm("method")),
    ast.StringTerm("PUT"),
),
```

则实际结果为

```
input[method] == "PUT"
```

这里的 `method` 由 `ast.VarTerm()` 函数创建，表示它是一个变量，也就是说此变量需要在其它地方先定义了才可以使用。

我们再看下数组，一般都是按索引取值的，如 `arr[0]`、`arr[1]` 之类，如

```
input[0] == "PUT"
```

正确写法为

```
ast.Equal.Expr(
    ast.RefTerm(ast.VarTerm("input"), ast.IntNumberTerm(0)),
    ast.StringTerm("PUT"),
),
```

这里用到了 `ast.IntNumberTerm()` 函数。

# 扩展阅读 

 * 示例中介绍的是一些基本用法，其中一些常用关键字这里并没有介绍，可参考 [官网文档](https://www.openpolicyagent.org/docs/latest/policy-reference/)

 * 对于赋值有两个操作符，一种是 `=` ，另一种是 `:=` ，后面的这种用法属于声明一个局部变量。同时在其声明的时候，变量名只能声明一次，在遇到对一个变量使用 `:=` 赋值值，编译器会检查这个变量是否已经被声明过。如果已声明过，则会编译出错，如 这个例子。

 * 系统针对每种数据类型提供了一些相应的 [内置函数][4]，同时也可以自定义函数。
 * 同一个规则名允许声明多次，只要其中任意一个满足条件即可，如 [这个例子][5]。

# 参考文档： 

* https://www.openpolicyagent.org/
* https://blog.openpolicyagent.org/
* https://pkg.go.dev/github.com/open-policy-agent/opa/rego
* https://www.openpolicyagent.org/docs/latest/integration/ #integrating-with-the-go-sdk

 [1]: https://en.wikipedia.org/wiki/Datalog
 [2]: https://play.openpolicyagent.org/p/qUkvgJRpIU
 [3]: https://www.openpolicyagent.org/docs/latest/policy-language/#some-keyword
 [4]: https://www.openpolicyagent.org/docs/latest/policy-reference/#built-in-functions
 [5]: https://play.openpolicyagent.org/p/59mnOAICAJ