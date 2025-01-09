---
title: Solana中如何解析指令
date: 2024-12-04T12:35:20+08:00
type: post
url: /posts/how-to-parse-instructions-in-solana
categories:
- 程序开发
- web3
tags:
- solana
- web3
---

开发过Solodity的同学都知道在合约开发中，不同指令对应的不同前端Endpoint(API接口)，这种开发模式特别的清晰且易维护。那在开发Solana合约时没有有对应的方法呢？

# Solana开发方式

开发 Solana 合约，一般分 Native 和 Anchor 框架开发。

 `Native` 主要是开发者通过SDK 手动实现所有业务逻辑。 这种模式一般对开发者要求比较高，除了需要了解相关概念外，最重要的还需要知道对应的SDK实现，如PDA账户的创建。

`Anchor框架` 推荐使用，只需要一些宏即可以实现一些逻辑，不需要用户关心底层实现。这种开发方式对于指令的处理基本与Solidity中一致，开发者只要搞明白了基本用法就可以了。

下面主要讲一下在 Native 这种方式下，如何实现指令或附加数据的解析。

如果你对 `指令`这个概念不太理解的话，可以将其视为路由。其类于似在mvc开发中控制器路由，如 `/user/info`、 `/user/base`、`/user/changepwd` 之类。

# 示例介绍

我们先看一个在 https://beta.solpg.io 网站上创建的一个 Native 示例

![image-20250109120746106](https://blog--static.oss-cn-shanghai.aliyuncs.com/uploads/2025/image-20250109120746106.png)

```rust
/// Define the type of state stored in accounts
#[derive(BorshSerialize, BorshDeserialize, Debug)]
pub struct GreetingAccount {
    /// number of greetings
    pub counter: u32,
}

// Declare and export the program's entrypoint
entrypoint!(process_instruction);

// Program entrypoint's implementation
pub fn process_instruction(
    program_id: &Pubkey, // Public key of the account the hello world program was loaded into
    accounts: &[AccountInfo], // The account to say hello to
    _instruction_data: &[u8], // Ignored, all helloworld instructions are hellos
) -> ProgramResult {
    msg!("Hello World Rust program entrypoint");

    // Iterating accounts is safer than indexing
    let accounts_iter = &mut accounts.iter();

    // Get the account to say hello to
    let account = next_account_info(accounts_iter)?;

    // The account must be owned by the program in order to modify its data
    if account.owner != program_id {
        msg!("Greeted account does not have the correct program id");
        return Err(ProgramError::IncorrectProgramId);
    }

    // Increment and store the number of times the account has been greeted
    let mut greeting_account = GreetingAccount::try_from_slice(&account.data.borrow())?;
    greeting_account.counter += 1;
    greeting_account.serialize(&mut *account.data.borrow_mut())?;

    msg!("Greeted {} time(s)!", greeting_account.counter);

    Ok(())
}
```

首先通过宏 `entrypoint!(process_instruction)` 声明了合约入口函数为  `process_instruction`，它是整个合约程序的的入口，所有进入合约内部的逻辑必须经过这个函数才可以进入，它类似 C 语言中的 `main()`函数。

对于 `process_instruction` 函数一共有三个参数，其参数个数以及位置是固定不可变的。你不能对参数做任何修改，如参数个数改变，或参数位置的调整都是不被允许的，其主要受限于 `entrypoint` 宏的实现 https://docs.rs/solana-program/2.1.7/solana_program/macro.entrypoint.html

```rust
#[macro_export]
macro_rules! entrypoint {
    ($process_instruction:ident) => {
        /// # Safety
        #[no_mangle]
        pub unsafe extern "C" fn entrypoint(input: *mut u8) -> u64 {
            let (program_id, accounts, instruction_data) = unsafe { $crate::deserialize(input) };
            match $process_instruction(program_id, &accounts, instruction_data) {
                Ok(()) => $crate::SUCCESS,
                Err(error) => error.into(),
            }
        }
        $crate::custom_heap_default!();
        $crate::custom_panic_default!();
    };
}
```

**参数解释**

`program_id` 程序ID，也称为合约ID，类似于Solidity里的合约地址。这里是一个 Pubkey类型，在前端 Typescript 脚本里一般使用 `new PublicKey(程序ID)`  声明

`accounts` 是一个 AccountInfo 数组，说明是多个账户

`instruction_data` 指令+数据，这里指函数标识和参数，它是一个 [u8] 类型，说明所有数据都是被序列化后的数据，因此需要先解析才能使用

在官方提供的示例中并没有使用到这个参数，因此命名为 `_instruction_data`，它是RUST中的一种用法，表示变量暂时用不到，编译器识别到这种用法就不会报错。

同时官方提示的单元测试示例中，前端调用代码也并不有传递这个参数，以下是 `native.test.ts` 文件的代码

```ts
// Create greet instruction
// 这里并没有传递第三个参数，因为合约里并没有使用指令参数
const greetIx = new web3.TransactionInstruction({
  keys: [
    {
      pubkey: greetingAccountKp.publicKey,
      isSigner: false,
      isWritable: true,
    },
  ],
  programId: pg.PROGRAM_ID,
});

// Create transaction and add the instructions
const tx = new web3.Transaction();
tx.add(createGreetingAccountIx, greetIx);
```

下面我们添加这个参数，并将其解析成不同指令且根据指令实现不同的逻辑。

# 指令解析

实现原理也很简单，第一个字节用来标识函数序号，因此我们只需要解析首个字节即可；如果有参数的话，接着解析后面的字节就可以了。

下面示例中用 0 表示 Increment，用 1 表示 Decrement。

这里首先将函数里的参数名 `_instruction_data` 改为 `instruction_data`, 然后在函数里添加以下代码

```rust
let (instruction_discriminant, instruction_data_inner) = instruction_data.split_at(1);
match instruction_discriminant[0] {
    0 => {
        msg!("Instruction: Increment");
				// TODO
    },
    1 => {
        msg!("Instruction: Decrement");
        // TODO
    }
    _ => {
        msg!("Error: unknown instruction")
    }
}
Ok(())
```

参数 `instruction_data` 是一个 `u8` 切片类型，通过 `instruction_data.split_at()` 函数将里面的字节元素分割成两部分：

- **第 1 个字节** 放入切片 `instruction_discriminant`变量中，此时这个切片变量只有一个元素，直接通过instruction_discriminant[0] 读取 

- **剩余的所有字节** 放入切片 `instruction_data_inner`， 这里存放的是附加数据

`instruction_discriminant[0]` 是从拆分后的数据中取出的第一个元素，用来表示指令类型。

> 这里 `instruction_discriminant` 是一个切片类型，它里面的元素是字节类型。如果通过 `instruction_data.split_at(2)` 赋值的话，则切片将有两个元素，此时可以通过 `instruction_discriminant[0]` 和 `instruction_discriminant[1]` 分别读取出来这两个值。

剩下的就是根据指令进行不同的业务处理了，同时对应的前端代码也要指定这个参数的值。

```ts
// Create greet instruction
const greetIx = new web3.TransactionInstruction({
  keys: [{...}],
  programId: pg.PROGRAM_ID,
  data: Buffer.from([0x0]), // 传递 instruction_data 指令
});
```

以上就是整个解析指令的过程，本质就是对二进制数据的读取操作。

接着我们看一下当指令附带有数据(函数包含参数)的时候，应该如何处理。

# 指令数据解析

还是同样的方法，直接解析数据。

这里假设指令 `1` 包含附加数据，这个数据为一个 `i32`类型的数字，这里特别指明一下数据类型，是因为数据解析需要知道不同类型的内存布局。

```rust
match instruction_discriminant[0] {
  ...
	1 => {
        msg!("Instruction: Decrement");
        // 解析附加数据
        let (data, _) = instruction_data_inner.split_at(4); // i32 类型占用 4 字节
        let value = i32::from_le_bytes(data.try_into().unwrap()); // 将字节数据转换为整数，采用小端
        msg!("Data: {}", value);
        // TODO
  }
  ...
}
```

这里首先将参数类型 `i32`对应的字节大小读取出来即可，而由于这个参数值占用 4 个字节，无法像单字节那样识别，因此还需要将这 4 个字节通过 `i32::from_le_bytes()`函数转换成整数。

这样就实现了参数解析，同样前端也要做相应的调整

```ts
import { web3 } from '@solana/web3.js';

const instructionDiscriminant = 0x1;  // 指令字节（例如：Decrement）
const value = 42;  // 附加数据（假设是一个 i32 整数）

// 将附加数据转换为 4 字节小端格式（i32）
const dataBuffer = Buffer.alloc(5);  // 总共 5 字节（1 字节指令 + 4 字节附加数据）
dataBuffer.writeUInt8(instructionDiscriminant, 0);  // 写入指令字节
dataBuffer.writeInt32LE(value, 1);  // 从第 1 字节开始写入 4 字节的数据，使用小端格式

// 创建 TransactionInstruction
const greetIx = new web3.TransactionInstruction({
  keys: [{...}],
  programId: pg.PROGRAM_ID,
  data: dataBuffer,  // 传递包含指令和数据的 Buffer
});
```

可以看到，通过 Native 方式开发时，解析指令以及附加数据的方法及原理。以上全是手动来实现的，理解这些需要了解数据结构内存总局，后端和前端都要知道有哪些参数类型以有对应内存占用大小，两个端要保持完全一致才可以。

不过平时在开发中，我们一般不采用这种手动解析指令的复杂用法。而是采用一些序列号库直接实现，如以下是来自官方[favorite](https://github.com/solana-developers/program-examples/blob/main/basics/favorites/native/program/src/processor.rs#L9-L15) 的实现代码

```rust
use solana_program::{
    account_info::AccountInfo, 
    entrypoint::ProgramResult, 
    pubkey::Pubkey,
};

use crate::instructions::{create_pda::*, get_pda::*};
use crate::state::Favorites;
use borsh::{BorshDeserialize, BorshSerialize};

#[derive(BorshDeserialize, BorshSerialize)]
pub enum FavoritesInstruction {
    CreatePda(Favorites),
    GetPda,
}

pub fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    let instruction = FavoritesInstruction::try_from_slice(instruction_data)?;

    match instruction {
        FavoritesInstruction::CreatePda(data) => create_pda(program_id, accounts, data),
        FavoritesInstruction::GetPda => get_pda(program_id,accounts),
    }?;

    Ok(())
}
```

这里先对指令集用一个 `enum` 类型来表示，同时使用宏为 `FavoritesInstruction` 枚举自动实现 `BorshDeserialize` 和 `BorshSerialize` 这两个`trait`，这样就可以直接调用  `FavoritesInstruction::try_from_slice` 这个函数来判断指令，同时在 match 条件语句中还可以直接解析参数值，非常的方便，在开发中几乎全是这种用法。

同时对应的前段代码也是采用类似的序列化库，只要保证数据结构以及字段类型与后端保持一致即可。

以下是后端与前端数据结构声明

后端(未含指令) https://github.com/solana-developers/program-examples/blob/main/basics/favorites/native/program/src/state.rs#L4-L7

前端 https://github.com/solana-developers/program-examples/blob/main/basics/favorites/native/tests/test.ts#L22-L57



# 总结

- 对于指令和数据的解析操作，就是将对应的字节大小转换成对应的实体数据
- 后端和前端数据类型要保持一致，或者是不同类型但只要内存布局一样即可。注意转换时大小端也要保持一致
- 开发中一般采用三库序列化库实现数据解析操作，并不需要手动解析数据
- 在 match 指令判断时，一般采用声明一个 enum 类型的数据结构来代替，它自身天生带有成员 tag 编号，如 `CreatePda(Favorites) `表示`0`，而 `GetPda` 则表示`1`



# 参考资料

- https://github.com/solana-developers/program-examples/blob/main/basics/counter/native/README.md
- https://github.com/solana-developers/program-examples/tree/main/basics/favorites/native/program/src