---
title: 彻底理解 ERC20 代币标准中的转账逻辑
date: 2024-11-07T11:55:42+08:00
type: post
toc: true
url: /posts/web3-solidity-erc20
categories:
  - web3
tags:
  - web3
  - solidity
  - erc20
---



在看 Solidity 文档 https://solidity-by-example.org/app/erc20/ 时，对于其中一段授权额的编码逻辑有点不明白，经过一翻查找资料才算彻底搞明白它的操作逻辑，这里特意将其记录一下。

# ERC20代币标准

在ERC20代币标准(https://eips.ethereum.org/EIPS/eip-20) 中定义了一系列的接口方法
```solidity
interface IERC20 {
  function totalSupply() external view returns (uint256); // 返回代币总供应量。
  function balanceOf(address account) external view returns (uint256); // 查询某个账户的代币余额。
  function transfer(address recipient, uint256 amount) external returns (bool); // 从调用者账户向其他地址转移代币。
  function allowance(address owner, address spender) external view returns (uint256); // 查询某地址被授权从另一个账户转移的额度。
  function approve(address spender, uint256 amount) external returns (bool); // 授权某地址从调用者账户转移一定数量的代币。
  function transferFrom(address sender, address recipient, uint256 amount) external returns (bool); // 由授权地址代表其他地址转移代币。
}
```
除此之外还有两个事件，虽然不是必须的，但还是强烈建议实现，方便以后排查错误。
注意上面的接口并不表示一定全部实现，需要根据场景来做判断。有些函数如 `totalSupply() `，如果使用声明变量 `uint256 public totalSupply`; 的方式的话，将自动生成对应的getter函数,因此就没有必要手动再实现了。

# 转账方式

在ERC20标准中，转账有两种方式：
- transfer 转账
常见的转账方式，适应于普通场景，假设 `账户A` 转币给 `账户B`，调用这个底层函数会直接从调用者的账户中扣除代币，并将其转账到指定的接收者。
- transferFrom 转账
这是另一种转账方式，通常用于需要授权的情况下，允许第三方代币持有者转账代币。此方法需要先通过 approve 授权一个代理账户（spender），表示代理可以从发送者账户中提取一定数量的代币（下方称为 `X`）。

大概步骤：
1. 首先 `账户A`给合约`账户X`一定的授权额度(最高)，表示 `合约X` 可以从 `A账户`里划走 `amount` 数量币，这个由 `账户A` 主动调用函数 `approve()`完成
2. 然后合约 `账户X` 才可以将币转给 `账户B`, 当然转账前必须检查授权额度是否足够才可以，如果足够则对两个账户的币进行加减，这个由`合约X` 调用函数 `transferFrom` 完成

可以看到相比普通转账，transferFrom 转账多了一个中间代理人而已，其它并没有什么不同。

以下主要对授权转账方式中的疑问做分析。

# 授权转账疑问

这里涉及的两个函数见 https://solidity-by-example.org/app/erc20/

```solidity
contract ERC20 is IERC20 {
   	mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

		// 授权额度
    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount; // 注意这一行
        emit Approval(msg.sender, spender, amount);
        return true;
    }

		// 转账
    function transferFrom(address sender, address recipient, uint256 amount)
        external
        returns (bool)
    {
        allowance[sender][msg.sender] -= amount;  // 注意这一行与 approve 函数里顺序不一样
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }
}

```

发现在 `approve` 函数里实现是 

```solidity
allowance[msg.sender][spender] = amount;
```

而在 `transferFrom` 函数里却是

```solidity
 allowance[sender][msg.sender] -= amount;
```

两者不一样，这是怎么回事呢？

要想彻底理解这一块，还得站在调用者视角来理解才行。

首先对于 `approve(address spender, uint256 amount)` 函数的调用是由 `账户A`  来完成的（关键）。它有两个参数，第一个参数 `spender` 表示授权给谁可以从账户上划走币，这里肯定是 `合约X`无疑了; 第二个参数 `amount` 则表示币的数量。 函数里的 `msg.sender` 就是 `账户A` 的地址，而 `spender` 则表示`合约X`的地址，这时授权语句

```solidity
allowance[msg.sender][spender] = amount;
```

就变成了

```solidity
allowance[账户A][合约X] = amount;
```

接着我们再看一下` transferFrom(address sender, address recipient, uint256 amount)` 函数，三个参数分别表示 `币的支付账户(A)`、`币的接收账户(B)`、`支付币的数量`。而对于这个函数的调用则是由 `合约X` 来完成的（关键）。于是函数里的 `sender` 就是`账户A`，而 `msg.sender` 就是 `合约X`，这时授权语句

```solidity
allowance[sender][msg.sender] -= amount; 
```

就变成了

```solidity
allowance[账户A][合约X] = amount;
```

可以看到它们执行的是同一个地址，这样一来对于上面两个变量的位置为什么不是一样的问题，就算搞明白了。

因此只要搞懂函数(指令)的当前调用者是谁是非常重要的。



# 总结

转账逻辑共两步：

1. 支付人给合约一定的授权额度，对应函数 `approve()`
2. 合约执行转账逻辑，对应函数 `transferFrom()`，记得要检查授权额是否足够

也就是说	

- approve 用于 **授权**，它给合约或第三方账户（如合约X）设置了一个代币转移的上限额度。**账户A** 调用 approve 授权 **合约X** 可以从其账户中转移最多 amount 数量的代币。

- transferFrom 用于 **代币转移**，它允许合约或第三方账户（如合约X）在获得授权的情况下，从 **账户A** 转移代币到 **账户B**。此操作必须检查授权额度是否足够，并且转移代币。

# 其它

1. transferFrom 与 transfer 区别
对于接口中的 transfer 函数，它是基础的转账函数，它常用于直接转账，它的使用场景就是我们上面讲到的普通场景；

而 transferFrom 函数一般用于授权转账。它通常用于去中心化交易所（DEX）或其他需要第三方代币转账的场景。授权方通过调用 approve 函数授权代理人可以从自己的账户转移一定数量的代币，然后代理人可以调用 transferFrom 来完成代币的转账。

2. 参数命名 sender 与 spender 的区别
参数命名时注意 `sender` 与 `spender` 的区别，其中 `sender` 一般表示代币发送者（账户A）, 如
```ts
 function transferFrom(address sender, address recipient, uint256 amount)
```
这里 `sender` 是代币发送者， 表示将`账户A`的钱转给 `账户B`; 
而 `spender` 则表示授权方（合约X)，如
```ts
  function approve(address spender, uint256 amount) external returns (bool);
```
这里 `spender` 是授权方，表示将当前调用指令的账户A的钱授权给 `合约X`。

# 参考资料

- https://solidity-by-example.org/app/erc20/
- https://eips.ethereum.org/EIPS/eip-20