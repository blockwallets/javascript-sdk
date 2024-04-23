# JavaScript SDK

## 概述

功能丰富 的 USDT-TRC20(TRON)、TRX(TRON) 收款接口服务;

通过以 JavaScript 的形式完成一个完整的支付流程。

[接入文档](https://blockwallets.io/docs/api)

## 特性

- 特点

  - 钱包是跟用户绑定长期有效的；

  - 具有钱包余额系统自动归集、自动提现等功能；

- 稳定：公开可靠公链信息，可随时在区块浏览器实时查询，数据实时同步；

- 其他：后续将支持多链。

### 先决条件

在操作支付流程之前，您需要：

- 1.访问并 登录/注册 [用户后台](https://blockwallets.io/auth/signin)
- 2.在用户后台查看 API 接入必需的 [api-key](https://blockwallets.io/account/apiKeys)
  ![index](https://raw.githubusercontent.com/blockwallets/public/main/images/wallet/api-key-page.png)
- 3.下载仓库的源码 demo，点击 index.html，直接通过浏览器开打

## 支付流程

- 1.创建钱包
  - 因为钱包是跟用户是唯一绑定的，所以可以为每个用户创建一个唯一且长期有效的钱包地址；
  - 一个开发者拥有一个系统主钱包，可以管理多个子钱包，子钱包都是跟每个用户相关联的；
  - 钱包用来进行后续的交易操作；
  - 目前仅支持 Tron（波场）钱包的创建。
- 2.等待用户支付
  - 当用户要进行支付操作时，这个时候需要通过当前支付用户的钱包地址，进行实时查询当前最新的交易记录；
  - 通过轮询的方式间隔一定的时间进行查询；
  - 在这个等待用户支付的过程中，可能需要较长时间，因为每个用户的支付时间都会有所不同，在这里可以自定义一个等待的有效时间或一直等待；
- 3.查询交易结果
  - 以用户发起支付时的时间为查询起始时间，查询在这个时间之后是否有最新的交易收款记录；
  - 当有最新的收款记录，查看对应的交易金额，就是用户支付所对应的金额，这里对用户支付多少金额目前没做要求；
  - 最终就完成了一笔完整的支付。

### 支付选项

![index](https://raw.githubusercontent.com/blockwallets/public/main/images/wallet/index.png)

### 1. 创建钱包

#### ① 填入 用户平台获取的 [api-key](https://blockwallets.io/account/apiKeys)

#### ② 点击 `创建钱包` 按钮

#### ③ 选择支付方式

创建钱包 JavaScript 代码

```javascript
// 创建钱包
async function handleCreateWallet() {
  const apiKey = encodeURIComponent(apiKeyInput.value); // 获取当前apiKey输入框的值并进行URL编码

  // 校验 api-key 输入是否为空
  if (apiKey.length === 0) {
    alert('请先填写 api-key');
    return null;
  }

  const data = {
    block_category_id: 1 // 区块链类别ID(1:Tron ...)
  };
  const headers = {
    'Content-Type': 'application/json',
    'api-key': apiKey
  };

  // 创建钱包
  async function handleCreateWallet() {
    const apiKey = encodeURIComponent(apiKeyInput.value); // 获取当前apiKey输入框的值并进行URL编码

    // 校验 api-key 输入是否为空
    if (apiKey.length === 0) {
      alert('请先填写 api-key');
      return null;
    }

    const data = {
      block_category_id: 1 // 区块链类别ID(1:Tron ...)
    };
    const headers = {
      'Content-Type': 'application/json',
      'api-key': apiKey
    };

    try {
      const response = await fetch(URL, {
        method: 'POST',
        headers: headers,
        body: JSON.stringify(data)
      });

      // 返回原始错误信息
      if (!response.ok) {
        throw response;
      }

      // 获取创建成功之后返回的信息
      const resData = await response.json();
      const address = resData.data.result.address;

      // 将获取到新的钱包地址，动态填充到输入框中
      var inputAddressElement = document.getElementById('addressInput');
      // 设置新的 value
      inputAddressElement.value = address;

      // 记录输入值
      localStorage.setItem('apiKey', apiKey);
      localStorage.setItem('address', address);

      alert('钱包创建成功');
    } catch (error) {
      try {
        const errorData = await error.json();
        const errorMessage = errorData.error.message;

        alert(errorMessage);
      } catch (error) {
        alert('Network Error');
      }
    }
  }
}
```

### 支付中心

![index](https://raw.githubusercontent.com/blockwallets/public/main/images/wallet/detail.png)

### 2. 等待用户支付

#### ① 通过钱包地址方式

#### ② 通过二维码方式

说明

- 用户需要通过其他方式向此钱包进行支付
- 自动轮询 的方式等待用户支付

自动检测相关 JavaScript 代码

```javascript
// 检测最新支付状态定时任务
// 定时任务: 每 20 秒执行一次 handleNewTransaction 函数，以检测是否有新的成功交易。这保证了页面上的交易信息和支付状态是最新的
const timer = setInterval(() => {
  // 只有当支付状态为 false 时执行
  if (!payStatus) {
    handleNewTransaction();
  }
}, 20000); // 每20秒执行一次
```

### 3. 查询交易结果

#### ③ 手动刷新

- 点击页面的刷新按钮

刷新相关 javascript 代码

```javascript
// 异步函数，用于从服务器获取最近的交易记录并更新页面显示。此函数在用户点击“刷新”按钮时触发
// 最近10条交易记录
async function handleRefresh() {
  const params = {
    address: payInfo.address, // 钱包地址
    token_type: payInfo.tokenType, // 代币类型
    limit: 10 // 查询条数
  };
  const queryString = new URLSearchParams(params).toString();
  const url = `${URL}?${queryString}`;
  const headers = new Headers({
    'Content-Type': 'application/json',
    'api-key': payInfo.apiKey
  });

  await fetch(url, { method: 'GET', headers: headers })
    .then((response) => {
      if (!response.ok) {
        throw new Error('网络请求失败');
      }
      return response.json();
    })
    .then((data) => {
      if (data.data.results) {
        const transactions = data.data.results;

        displayTransactions(transactions);
      } else {
        const container = document.getElementById('transactionsTable');
        container.innerHTML = `
                               <hr />
                                <p class="py-8">暂无交易数据</p>
                               `;
      }
    })
    .catch((error) => {
      console.error('加载数据时发生错误:', error);
    });
}
```

#### ④ 最新交易列表

- 获取到最新的交易信息

最新交易相关 JavaScript 代码

```javascript
// 获取最新交易记录
// 检测新交易并更新状态
async function handleNewTransaction() {
  const params = {
    address: payInfo.address,
    token_type: payInfo.tokenType,
    start_time: startTime
  };
  const queryString = new URLSearchParams(params).toString();
  const url = `${URL}?${queryString}`;
  const headers = new Headers({
    'Content-Type': 'application/json',
    'api-key': payInfo.apiKey
  });

  try {
    const response = await fetch(url, {
      method: 'GET',
      headers: headers
    });
    if (!response.ok) {
      throw new Error('网络请求失败');
    }
    const data = await response.json();
    if (data.data.results) {
      const transactions = data.data.results;

      const newSuccessfulTransaction = transactions.some(
        (transaction) => transaction.category === 'income'
      );
      if (newSuccessfulTransaction) {
        // 支付成功
        payStatus = true; // 修改支付状态

        // 重新渲染支付状态信息
        const payStatusHtml = document.getElementById('payStatus');
        payStatusHtml.innerHTML = `
                                    <div id='payStatus' class="flex items-centers">
                                    <p class="py-2">支付状态：</p>
                                    <span class="text-green-500 py-2">已支付</span>
                                    </div>
                                `;

        // 重新更新交易列表信息
        await handleRefresh();
      }
    }
  } catch (error) {
    console.error('获取新交易时出错: ', error);
  }
}
```

### 结语

要想运行完整代码示例，请下载仓库源码
