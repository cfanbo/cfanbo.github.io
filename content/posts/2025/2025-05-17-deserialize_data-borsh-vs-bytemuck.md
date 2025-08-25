---
title: 零拷贝bytemuck与 borsh
date: 2025-05-17T17:59:20+08:00
type: post
url: /posts/deserialize-data-borsh-vs-bytemuck
toc: true
categories:
- 程序开发
tags:
- bytemuck
- zerocopy
- 零拷贝
- borsh
---

在上一篇博文《[深入理解 Serde、Bincode 与 Borsh 的关系与区别](https://blog.haohtml.com/posts/serde-vs-bincode-vs-borsh-in-the-rust/)》介绍了常用的几种解析二进制数据的方法，主要有 bincode 与 borsh, 并提到过在区块链领域里一般推荐使用 borsh 解析数据。但随着合约的开发使用borsh的地方越来越多，会经常遇到提示超出 4K Stack 大小的错误。这是因为在solana里，虚拟机 sbf 限制了一个合约最大允许使用的statck大小上限为 4k。尽管我们使用完一个大变量通过一些方法，如变量作用域、通过Box将内存移动到heap、或手动drop立即释放内存。但仍有些场景是没有采用这种办法的，这时应该如何办呢？

如果经常看一些优秀的开源项目的话，会发现有一个 `bytemuck` 的crate，它是一个 `zerocopy` 库，可以避免内存复制带来的开销，加速解析数据速度，这里给出一个测试代码

```rust
use borsh::{BorshDeserialize, BorshSerialize};
use bytemuck::{Pod, Zeroable};
use solana_program::pubkey::Pubkey;
use std::time::Instant;

/// -------- 零拷贝结构 (定长布局) --------
#[repr(C, packed)]
#[derive(Clone, Copy, Pod, Zeroable, Debug)]
pub struct AccountZC {
    pub lamports: u64,  // 8字节
    pub data: [u8; 32], // 32字节
    pub owner: Pubkey,  // 32字节
}

/// -------- Borsh 结构 --------
#[derive(BorshSerialize, BorshDeserialize, Clone, Debug)]
pub struct AccountBorsh {
    pub lamports: u64,
    pub data: [u8; 32],
    pub owner: Pubkey,
}

fn main() {
    // 模拟数据
    let lamports: u64 = 123456789;
    let data: [u8; 32] = [7u8; 32]; // 32字节全是 7
    let owner = Pubkey::new_unique();

    // ---------- 用 Borsh 序列化测试数据 ----------
    let account_borsh = AccountBorsh {
        lamports,
        data,
        owner,
    };
    let serialized_raw = borsh::to_vec(&account_borsh).unwrap();
    println!("raw序列化长度 = {}", serialized_raw.len());
    println!("serialized_raw首地址 = {:p}", serialized_raw.as_ptr());

    // ---------- 1. 用 Borsh 反序列化解析 ----------
    let start = Instant::now();
    let parsed_borsh = AccountBorsh::try_from_slice(&serialized_raw).unwrap();
    println!("零拷贝解析结果 = {:?}", parsed_borsh);
    println!("反序列化耗时: {:?}", start.elapsed());
    println!("Borsh struct首地址 = {:p}", &parsed_borsh);

    // ---------- 2. 零拷贝解析 ----------
    // 因为 AccountZC 是定长布局，可以直接按字节存放
    let start = Instant::now();
    assert!(serialized_raw.len() >= std::mem::size_of::<AccountZC>());
    let acc: &AccountZC = bytemuck::from_bytes(&mut serialized_raw.as_ref());
    println!("零拷贝解析结果 = {:?}", acc);
    println!("反序列化耗时: {:?}", start.elapsed(),);
    println!("Bytemuck struct首地址 = {:p}", acc);
}
```

输出

```SHELL
raw序列化长度 = 72
serialized_raw首地址 = 0x134008800

零拷贝解析结果 = AccountBorsh { lamports: 123456789, data: [7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7], owner: 11157t3sqMV725NVRLrVQbAu98Jjfk1uCKehJnXXQs }
反序列化耗时: 41.791µs
Borsh struct首地址 = 0x16f648f68

零拷贝解析结果 = AccountZC { lamports: 123456789, data: [7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7, 7], owner: 11157t3sqMV725NVRLrVQbAu98Jjfk1uCKehJnXXQs }
反序列化耗时: 11.959µs
Bytemuck struct首地址 = 0x134008800
```

可以看到解析速度足足相差了三倍之多（多次执行可能不一样），想想也是挺恐怖的。不过这不是本文的重点，我们主要看一下它是如何实现零拷贝的。



# borsh 解析原理

对于 borsh 解析数据时，它的主要步骤大概是这个样子的。

1. **初始化输入游标**
    Borsh 会把 `&[u8]` 封装成一个 `&mut &[u8]` 或者类似的 `Cursor`，这样可以在逐字段读取时维护当前偏移位置。

2. **逐字段调用 `BorshDeserialize`**
   每个字段都要实现 `BorshDeserialize`。

   - `u64`：固定 8 字节，小端序读取
   - `[u8; 32]`：固定长度数组，依次复制 32 字节
   - `Pubkey`（通常 `[u8; 32]`）：也是固定长度数组，复制 32 字节

   这里每次读取 **不会一次性复制到 struct 内存中**，而是把解析好的字段值返回出来。

3. **构造 struct 实例**
       在 Rust 层面，Borsh 会调用 `AccountBorsh { lamports, data, owner }` 来组装一个新的实例。
       也就是说，它是采用 **Rust 风格的安全逐字段构造**。

4. **检查剩余数据**
   如果反序列化完 struct 后，输入切片中仍有未消费的数据，Borsh 会返回错误，保证字节流和 struct 严格匹配。

可以看到它完全与我们平时开发时采用的方法是一样的，先读取原来每个字段的值，再构建一个新struct，并为每一个字段赋值，这一点很容易理解的。

# 零拷贝 bytemuck

我们再看一下 bytemuck 这个 zerocopy 库，它是如何实现的。

解析步骤：

1. 首先检查要解析数据的大小是否大于或等于解析目标 struct (这里是 AccountZC) 的大小，以保证解析一定成功。也正因为如此，我们在调用 `bytemuck::from_bytes()`时，并不需要考虑解析panic的情况

2. 直接将字节切片 serialized_raw 的首地址强制转换成 struct 类型的引用

   ```rust
   let ptr: *const T = bytes.as_ptr() as *const T; // 只读
   let reference: &T = &*ptr;
   ```

   注意，这里并没有进行任何数据的复制！没有复制！没有复制！

3. 返回类型引用 `&T` 或 `&mut T`

看到了吧，相比borsh而言， bytemuck 根本就不存在对任何数据的复制操作，它只是将这个对象的地址改成了解析数据的首地址，也就是说，struct 的首地址就是 serialized_raw 的首地址，上面的输出 `serialized_raw首地址 = Bytemuck struct首地址 = 0x134008800` 也证明了这一点。

总结如下：

- bytemuck **不申请整个结构体大小的内存**

- 只“申请”或使用了一个指针大小的变量来指向原内存

- 访问字段就是直接读写原始字节，不做复制

这也正是为什么上面显示 bytemuck 解析速度要比 borsh 快三倍的原因。

# 总结

那么既然零拷贝如何高效，是不是可以直接将 bytemuck 全部用来代替 borsh呢？

答案是 **不行的**。

主要原因是  bytemuck 只能用在数据类型长度固定的场景，而对于长度非固定的数据类型，如 String 、Vec 之类的则不行的。像上面 data 字段，它的长度是已知为 8 的，所以可以直接使用 bytemuck的。

另外对于要使用bytemuck 的struct ，需要满足几个条件：

1. 要实现 Pod 和 Zeroable 这两个trait
2. 要保证内存对齐要求 `repc(c)`，而对于` repc(packed)` 则表示告诉编译器**取消字段的默认对齐填充（padding）**，结构体紧凑排列，它是一个可选项。一般推荐两者一起使用。

希望通过这一篇博文，方便大家更好的理解 bytemuck 的原理及它的使用场景。
