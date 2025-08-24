---
title: 深入理解 Serde、Bincode 与 Borsh 的关系与区别
date: 2025-04-22T20:32:20+08:00
type: post
url: /posts/serde-vs-bincode-vs-borsh-in-the-rust
toc: true
categories:
- 程序开发
- web3
tags:
- web3
- serde
- bincode
- borsh
---

在Rust开发中，无论是构建网络服务、存储数据还是开发区块链程序，**序列化（Serialization）和反序列化（Deserialization）**都是不可或缺的操作。序列化是将内存中的数据结构(struct)转换成字节序列或者其他格式(JSON, vec<u8>)，以便存储或传输；反序列化则是将字节序列还原成原来的数据结构。

Rust 生态中常用的序列化工具包括 **Serde**、**Bincode** 和 **Borsh**。初学者在阅读文档或实际开发中可能会发现它们名字都很熟悉，但它们的定位、使用方式和特点却不完全相同。

本文将系统梳理三者的关系、差异和使用场景。

## Serde

**Serde（Serialization / Deserialization）** 是 Rust官方生态最流行的序列化框架。它提供了一种 **抽象接口**，让你可以将 Rust 类型序列化为多种格式，而不关心底层具体实现。

### 特点

- **通用性强**：支持 JSON、YAML、TOML、Bincode 等多种格式。
- **灵活**：可以自定义序列化逻辑。
- **宏支持**：通过 `#[derive(Serialize, Deserialize)]` 自动生成序列化代码。
- **安全性**：类型系统保证序列化和反序列化操作类型安全。

### 示例

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    age: u32,
}

fn main() {
    let user = User { name: "Alice".to_string(), age: 30 };
    let json = serde_json::to_string(&user).unwrap();
    println!("JSON: {}", json);
}
// Output:
// JSON: {"name":"Alice","age":30}
```

> 注意：Serde 本身不提供具体的存储格式，它是框架，依赖具体格式库（如
> `serde_json`、`bincode`）来实现序列化。

## Bincode

**Bincode** 是 Serde 生态下的一个**高性能**二进制序列化库，专注于
**速度和空间效率**。

### 特点

- **二进制格式**：比 JSON 更节省空间，更适合网络传输或文件存储。
- **高性能**：序列化和反序列化速度都非常快。
- **支持动态类型**：可以序列化 `Vec`、`String` 等动态长度类型。
- **依赖 Serde**：必须实现 `Serialize` 和 `Deserialize`。

### 示例

```rust
use serde::{Serialize, Deserialize};
use bincode;

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String, // String 为非固定长度数据类型
    age: u32,
}

fn main() {
    let user = User { name: "Alice".to_string(), age: 30 };
    let bytes = bincode::serialize(&user).unwrap();
    println!("Bincode bytes: {:?}", bytes);

    let user2: User = bincode::deserialize(&bytes).unwrap();
    println!("Deserialized: {:?}", user2);
}
// Output:
// Bincode bytes: [
//	5, 0, 0, 0, 0, 0, 0, 0, // 单独存储 name字段值 长度
//	65, 108, 105, 99, 101,  // Alice
//  30, 0, 0, 0 						// 30u32
// ]
// Deserialized: User { name: "Alice", age: 30 }
```

- Bincode 会将每个字段按照固定顺序写入字节数组。
- 对于动态类型（如 `Vec` 和 `String`），会额外存储**长度信息**。
- 在大多数非区块链 Rust 项目中，Bincode 是非常高效且通用的选择。

## Borsh

**Borsh（Binary Object Representation Serializer for Hashing）** 是 Solana、NEAR
等区块链项目常用的二进制序列化库(主要用于链上)。

Borsh 的设计目标是：

1. **确定性**：序列化结果固定，便于哈希计算和链上验证。
2. **高性能**：优化链上账户存储访问，减少 CPU 和存储消耗。
3. **紧凑存储**：对动态数组长度使用固定字节（通常是 u32），节省空间。

### 特点

- **不依赖 Serde**：Borsh 自己实现序列化接口，与bincode 的主要区别。
- **固定字段顺序**：序列化结果严格按照字段声明顺序，避免不同版本或平台不一致。
- **动态字段支持有限**：支持 `Vec` 和 `String`，但**长度信息**固定为 **u32**。
- **链上友好**：在 Solana 中，每个账户的数据大小、序列化结果都是可预测的，方便做
  hash 或签名验证。

### 示例

```rust
use borsh::{BorshDeserialize, BorshSerialize};

#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct User {
    name: String,
    age: u32,
}

fn main() {
    let user = User {
        name: "Alice".to_string(),
        age: 30,
    };
    let bytes = borsh::to_vec(&user).unwrap();
    println!("Borsh bytes: {:?}", bytes);

    let user2 = User::try_from_slice(&bytes).unwrap();
    println!("Deserialized: {:?}", user2);
}

// Output:
// Borsh bytes: [5, 0, 0, 0, 65, 108, 105, 99, 101, 30, 0, 0, 0]
// Deserialized: User { name: "Alice", age: 30 }
```

- Borsh 更适合链上账户存储和状态更新，确保每个字段的位置、长度和顺序都是固定的。
- 不像 Bincode 那样可以依赖 Serde，因此链上无需额外宏或类型推导。

## Bincode 与 Borsh

通过上面的介绍，可以看到对于二进制序列化操作，我们的选择主要在 Bincode 与 Borsh之间，那么它们之间又有何区别呢？

下面我们从数据类型固定长度和非固定长度的情况看一下两者的区别。

### 数据类型固定长度

```rust
use borsh::{BorshDeserialize, BorshSerialize};
use serde::{Deserialize, Serialize};

#[derive(Debug, BorshSerialize, BorshDeserialize, Serialize, Deserialize)]
struct Point {
    x: u32,
    y: u32,
}

fn main() {
    let point = Point { x: 1, y: 2 };

    // 使用 Bincode 序列化
    let bincode_bytes = bincode::serialize(&point).unwrap();
    println!("Bincode bytes: {:?}", bincode_bytes);
    println!("Bincode total length: {}", bincode_bytes.len());

    // 使用 Borsh 序列化
    let borsh_bytes = borsh::to_vec(&point).unwrap();
    println!("Borsh bytes: {:?}", borsh_bytes);
    println!("Borsh total length: {}", borsh_bytes.len());
}
```

输出：

```shell
Bincode bytes: [1, 0, 0, 0, 2, 0, 0, 0]
Bincode total length: 8

Borsh bytes: [1, 0, 0, 0, 2, 0, 0, 0]
Borsh total length: 8
```

这时两者是一样的，没有任何区别。

### 数据类型长度非固定

我们再看一下在数据类型长度不固定的情况，如 Vec 和 String 数据类型。

这里是一个简单的数组 `vec![1u32, 2, 3]`，序列化后的字节如下：

```rust
use borsh::{BorshDeserialize, BorshSerialize};
use serde::{Deserialize, Serialize};

#[derive(Debug, BorshSerialize, BorshDeserialize, Serialize, Deserialize)]
struct Point {
    name: String, // 不固定大小
    x: u32,
    y: u32,
    numbers: Vec<u32>, // 不固定大小
}

fn main() {
    let point = Point {
        name: "Alice".to_string(),
        y: 20,
        x: 10,
        numbers: vec![1, 2, 3],
    };

    // Bincode 序列化
    let bincode_bytes = bincode::serialize(&point).unwrap();
    println!("Bincode bytes: {:?}", bincode_bytes);
    println!("Bincode total length: {}", bincode_bytes.len());

    // Borsh 序列化
    let borsh_bytes = borsh::to_vec(&point).unwrap();
    println!("Borsh bytes: {:?}", borsh_bytes);
    println!("Borsh total length: {}", borsh_bytes.len());
}
```

输出

```shell
Bincode bytes: [
  5, 0, 0, 0, 0, 0, 0, 0,  // name 长度 u64 [8字节]
  65, 108, 105, 99, 101,  // "Alice" [5]
  10, 0, 0, 0, 						// x [4]
  20, 0, 0, 0, 						// y [4]
  3, 0, 0, 0, 0, 0, 0, 0, // numbers 长度 u64 [8]
  1, 0, 0, 0, 						// [4]
  2, 0, 0, 0, 						// [4]
  3, 0, 0, 0							// [4]
]
Bincode total length: 41 // 8 + 5 + 4 * 2 + 8 + 4 * 3 = 41


Borsh bytes: [
  5, 0, 0, 0,  // name 长度 u32 [4字节]
  65, 108, 105, 99, 101,   // "Alice" [5]
  10, 0, 0, 0, // x [4]
  20, 0, 0, 0, // y [4]
  3, 0, 0, 0, // numbers 长度 u32 [4]
  1, 0, 0, 0, // [4]
  2, 0, 0, 0, // [4]
  3, 0, 0, 0// [4]
]
Borsh total length: 33 // 4 + 5 + 4 * 6 = 33
```

这里序列化操作都需要记录数据类型非固定长度的总字节大小，对于bincode
**默认**使用的是 u64 类型（占 8 字节）。而borsh则使用u32类型（占 4 字节）：

字段 `name: String` ，值为 `Alice`，长度是 `5`，则使用一个 u32或u64 类型存储值
`5`; 字段 `numbers: Vec<u32>`, 值 为`vec![1, 2, 3]`，长度是 `3`，则使用一个
u32或u64 类型存储值 `3`。

可以看到，如果一个struct里存在数据类型非固定长度字段的话，则使用 bincode
要比比borsh多占用 4 个字节。

这也是为什么在 solana
区块链领域，多数情况下都采用borsh序列化的主要原因，因为链上大部分数据都在几M以内，使用borsh要节省一些内存，要知道链上空间是十分昂贵的。

> 如果链上数据过大，使用borsh将解析失败，但bincode则正常。

## 三者对比

| 特性         | Serde                           | Bincode              | Borsh                    |
| ------------ | ------------------------------- | -------------------- | ------------------------ |
| 类型         | 框架                            | 序列化格式           | 序列化格式               |
| 依赖         | —                               | ✅ Serde             | ❌ 独立实现              |
| 序列化方式   | 多种格式（JSON、Bincode、YAML） | 二进制               | 二进制（链上友好）       |
| 动态字段支持 | ✅                              | ✅                   | ✅（长度用 u32）         |
| 固定字段顺序 | 可选                            | 默认                 | ✅ 必须                  |
| 使用场景     | 通用序列化, JSON/TOML等解析     | 高性能网络、文件存储 | 区块链账户存储、状态验证 |

> 总结：
>
> 1. Serde 是一个序列化框架，提供统一接口，在rust里简称 `trait`
> 2. Bincode 与 borsh 都是二进制序列化
> 3. Bincode 使用 Serde 提供高效二进制，它依赖于 Serde
> 4. Borsh 独立实现，更加轻量且链上友好, 区块链上首选

## 总结

1. **Serde** 是框架，不负责具体字节格式，提供抽象接口。
2. **Bincode** 是高性能二进制格式，依赖 Serde，支持动态长度类型。
3. **Borsh** 是链上序列化格式，不依赖
   Serde，字段顺序固定，便于哈希计算和账户存储。
4. **选择依据**：性能、可读性、链上需求、数据格式一致性。

通过上面的介绍，想必大家对这三个crate有了一定的了解，对于开发中应该如何选择它们呢，不妨参考以下几个原则：

- **链上开发（Solana / NEAR）**：Borsh，节省内存，字段顺序固定，方便哈希和状态验证。
- **普通 Rust 应用**：Bincode + Serde，高性能二进制，支持复杂数据结构。
- **跨语言、可读性要求高**：Serde + JSON，便于调试和日志记录。
