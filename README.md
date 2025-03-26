# Timelock 智能合约

基于 Stylus 开发的时间锁合约，用于延迟执行交易。

## 环境配置
```
export ARB_DEMO=0x6c49d46cf7267A3De0A698cab95792BF69c91aFC
export PRIVATE_KEY=0x.....
export ARB_RPC=https://sepolia-rollup.arbitrum.io/rpc
```

## 开发步骤

### 1. 安装依赖

在 `Cargo.toml` 中添加：
```toml
[dependencies]
sha3 = "0.10.8"
```

### 2. 部署合约

```bash
# 检查合约
cargo stylus check -e $ARB_RPC

# 部署合约
cargo stylus deploy --endpoint=$ARB_RPC --private-key=$PRIVATE_KEY
```

### 3. 合约初始化

```bash
# 设置已部署的合约地址
export TIMELOCK=你的合约地址

# 初始化合约
cast send $TIMELOCK "initialize()" -r=$ARB_RPC --private-key=$PRIVATE_KEY

# 验证所有者
cast call $TIMELOCK "owner()" -r=$ARB_RPC
```

### 4. 存入测试 ETH

```bash
# 存入 1 wei
cast send $TIMELOCK "deposit()" --value 1wei -r=$ARB_RPC --private-key=$PRIVATE_KEY

# 查看余额
cast balance $TIMELOCK -r=$ARB_RPC
```

### 5. 添加延时交易

```bash
# 创建地址
cast wallet new

# 获取当前区块时间戳,  1742979842
cast block latest -r=$ARB_RPC

# 通过合约转账 to.call{value: msg.value}("")
# 添加交易到队列 (timestamp = 当前时间 + 300秒)
cast send $TIMELOCK "queue(address,uint256,string,bytes,uint256)" \
    接收地址 \
    转账金额 \
    "" \
    0x \
    时间戳 \
    -r=$ARB_RPC --private-key=$PRIVATE_KEY
    
cast send $Timelock "queue(address,uint256,string,bytes,uint256) "0x3a4848aa2f4f1D0F497C56c630831DAc7De46c43 1 "" 0x 1742979902 -r=$ARB_RPC --private-key=$PRIVATE_KEY
```

### 6. 执行交易

```bash
# 执行已排队的交易
cast send $TIMELOCK "execute(address,uint256,string,bytes,uint256)" \
    接收地址 \
    转账金额 \
    "" \
    0x \
    时间戳 \
    -r=$ARB_RPC --private-key=$PRIVATE_KEY

cast send $Timelock "execute(address,uint256,string,bytes,uint256)" 0x3a4848aa2f4f1D0F497C56c630831DAc7De46c43 1 "" 0x 1742979902 -r=$ARB_RPC --private-key=$PRIVATE_KEY
```

### 调试工具

```bash
# 根据 selector 查找函数签名
cast 4byte 0x035fce5b
```

## 注意事项
- 确保执行交易时已经超过延迟时间
- 交易执行必须在 GRACE_PERIOD 期限内
- 只有合约所有者可以添加和执行交易