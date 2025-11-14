# cfdohpw

Cloudflare DoH proxy worker。

借助 Cloudflare CDN 平台中转加速任意 DoH (RFC8484) 流量。无需服务器和域名。

注：本项目代码已经改为代码 ES modules 格式

## 如何使用

Worker 的代码在项目根目录的 [index.js](https://github.com/IrineSistiana/cfdohpw/blob/main/index.js)。



### 不用软件只需浏览器点鼠标

可以用 Cloudflare 网页控制台部署 worker。Cloudflare 经常更新它的网页，此处省略一万字，请 Google，教程很多。

### 命令行

需要 Cloudflare 官方工具 [wrangler](https://github.com/cloudflare/wrangler)。

**部署步骤**
在终端中执行以下命令：

# 1. 安装 Wrangler CLI (如果还没安装)
npm install -g wrangler

# 2. 登录 Cloudflare 账户
wrangler login

# 3. 进入项目目录
cd doh-proxy

# 4. 部署
wrangler deploy

**使用方法**
部署成功后，你会得到一个类似 https://doh-proxy.your-account.workers.dev 的地址。

测试 GET 请求：

# 查询 example.com 的 A 记录
curl -H "accept: application/dns-message" \
  "https://doh-proxy.your-account.workers.dev/dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB"
测试 POST 请求：

echo -ne '\x00\x00\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x07example\x03com\x00\x00\x01\x00\x01' | \
  curl -X POST \
  -H "content-type: application/dns-message" \
  -H "accept: application/dns-message" \
  --data-binary @- \
  "https://doh-proxy.your-account.workers.dev/dns-query"


📝 额外建议
修改 endpointPath：建议改为难以猜测的路径，如：

const endpointPath = '/my-secret-dns-path-12345';
添加其他 DoH 提供商（可选）：

// Cloudflare DoH
const upstream = 'https://cloudflare-dns.com/dns-query';

// Quad9
const upstream = 'https://dns.quad9.net/dns-query';


## 如何避免所有人都能使用该 worker

虽然平常见到的 DoH 服务器的地址都是 `https://<server_addr>/dns-query`。但这个路径 `/dns-query` 其实是可以随意改的。

可以修改 index.js 中的 `endpointPath` 参数来改变该路径。

比如设定

```js
const endpointPath = '/dns-query-with-my-passwd-123456';
```

则服务器地址变成 `https://<server_addr>/dns-query-with-my-passwd-123456`。只有知道路径的用户才能使用该 worker。

## Credit

- [Cloudflare Workers](https://workers.cloudflare.com/)
