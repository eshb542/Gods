# DeFi API赎回流程快速入门指南

## 1. 环境配置与准备
在进行DeFi产品赎回操作前，必须完成基础环境搭建。开发者需重点完成以下核心配置：
- 安装Node.js 18.x或更高版本
- 配置Web3.js 1.8.2及以上版本
- 获取OKX Web3 API访问密钥

👉 [快速获取Web3开发资源](https://bit.ly/okx_welcome)

### 1.1 导入必要依赖库
通过npm安装以下核心依赖包：
```bash
npm install web3 @okxweb3/defi-sdk
```
配置环境变量时，请确保：
| 参数 | 类型 | 示例值 |
|------|------|--------|
| API_KEY | String | "your_api_key_here" |
| NETWORK | String | "ethereum-mainnet" |

### 1.2 初始化SDK实例
```javascript
const { DefiSDK } = require('@okxweb3/defi-sdk');
const defi = new DefiSDK({
  apiKey: process.env.API_KEY,
  network: process.env.NETWORK
});
```

## 2. 用户持仓查询操作
### 2.1 核心参数定义
必须提供以下查询参数：
```javascript
const params = {
  userAddress: "0x...user_wallet_address",
  protocolId: "aave_v3",
  chainId: 1
};
```

### 2.2 查询执行方法
```javascript
async function queryPositions() {
  try {
    const result = await defi.getPositions(params);
    console.log("用户持仓详情:", result);
  } catch (error) {
    console.error("查询失败:", error);
  }
}
```

### FAQ
**Q：查询时提示"Permission Denied"如何处理？**  
A：请检查API密钥权限配置，确保已开通对应链的访问权限。

**Q：如何获取支持的协议列表？**  
👉 [查看完整协议支持清单](https://bit.ly/okx_welcome)

## 3. 投资产品信息获取
### 3.1 产品参数查询
```javascript
const productParams = {
  productId: "UNI-V3-0x1f5",
  chainId: 1
};
```

### 3.2 产品详情获取
```javascript
async function getProductDetails() {
  try {
    const details = await defi.getProductInfo(productParams);
    console.table(details);
  } catch (error) {
    console.error("产品信息获取失败:", error);
  }
}
```

## 4. 赎回预估计算
### 4.1 计算参数设置
```javascript
const estimateParams = {
  userAddress: "0x...user_wallet_address",
  productId: "UNI-V3-0x1f5",
  amount: "1000000000000000000" // 1 ETH
};
```

### 4.2 执行预估计算
```javascript
async function calculateEstimate() {
  try {
    const estimate = await defi.getRedemptionEstimate(estimateParams);
    console.log(`预计赎回价值: ${estimate.value} USD`);
    console.log(`预计gas费用: ${estimate.gas} ETH`);
  } catch (error) {
    console.error("计算失败:", error);
  }
}
```

**Q：预估结果与实际赎回存在差异怎么办？**  
A：市场波动可能导致估算偏差，建议在交易前30秒内重新计算。

## 5. 赎回交易构建
### 5.1 授权数据生成
```javascript
async function generateAuthData() {
  try {
    const authData = await defi.generateRedemptionAuth(estimateParams);
    console.log("授权数据:", authData);
  } catch (error) {
    console.error("生成失败:", error);
  }
}
```

### 5.2 交易数据生成
```javascript
async function generateTxData() {
  try {
    const txData = await defi.generateRedemptionTransaction({
      ...estimateParams,
      authSignature: "0x...signature"
    });
    console.log("交易数据:", txData);
  } catch (error) {
    console.error("生成失败:", error);
  }
}
```

## 6. 交易签名与广播
### 6.1 本地签名方案
```javascript
const { signTransaction } = require('web3-eth-accounts');

async function signAndSend() {
  const signedTx = await signTransaction(txData, "0x...privateKey");
  try {
    const txHash = await defi.broadcastTransaction({
      chainId: 1,
      signedTx: signedTx.rawTransaction
    });
    console.log("交易哈希:", txHash);
  } catch (error) {
    console.error("广播失败:", error);
  }
}
```

### FAQ
**Q：如何监控交易状态？**  
👉 [交易状态实时查询工具](https://bit.ly/okx_welcome)

**Q：交易被链上拒绝如何处理？**  
A：建议检查gas价格设置（建议设置为baseFee + 20%），并查看智能合约日志。

## 7. 高级功能与优化
### 7.1 批量赎回优化
通过batchRedemption接口可实现多产品同时赎回：
```javascript
const batchParams = {
  userAddress: "0x...user_wallet_address",
  products: [
    { productId: "UNI-V3-0x1f5", amount: "1000000000000000000" },
    { productId: "AAVE-V3-DAI", amount: "2000000000000000000" }
  ]
};
```

### 7.2 动态Gas管理
实现自适应gas价格计算：
```javascript
async function getOptimalGas() {
  const { baseFee, priorityFee } = await defi.getGasRecommendation({ chainId: 1 });
  return {
    gasPrice: baseFee * 1.2,
    maxPriorityFeePerGas: priorityFee * 1.5
  };
}
```

## 8. 常见错误解决方案
| 错误代码 | 原因分析 | 解决方案 |
|---------|----------|----------|
| 403 Forbidden | API密钥无效 | 检查密钥有效性及网络配置 |
| 503 Service Unavailable | 链上服务中断 | 切换备用RPC节点 |
| 11002 Gas不足 | gas预估过低 | 增加gasLimit参数20% |

### FAQ
**Q：如何获取最新API版本更新？**  
👉 [订阅API更新日志](https://bit.ly/okx_welcome)

**Q：测试网环境如何配置？**  
A：将network参数改为对应测试链标识，如"ethereum-goerli"。