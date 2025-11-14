<div align="center">

# VLESS-XHTTP Proxy Server

[![npm version](https://img.shields.io/npm/v/@eooce/xhttp?style=flat-square)](https://www.npmjs.com/package/@eooce/xhttp)
[![npm downloads](https://img.shields.io/npm/dm/@eooce/xhttp?style=flat-square)](https://www.npmjs.com/package/@eooce/xhttp)
[![GitHub stars](https://img.shields.io/github/stars/eooce/serverless-xhttp?style=social)](https://github.com/eooce/serverless-xhttp)
[![GitHub forks](https://img.shields.io/github/forks/eooce/serverless-xhttp?style=social)](https://github.com/eooce/serverless-xhttp)
[![GitHub issues](https://img.shields.io/github/issues/eooce/serverless-xhttp)](https://github.com/eooce/serverless-xhttp/issues)
[![GitHub license](https://img.shields.io/github/license/eooce/serverless-xhttp)](https://github.com/eooce/serverless-xhttp/blob/main/LICENSE)


一个高性能的VLESS-XHTTP代理服务器，基于Node.js实现，支持主流客户端

</div>

## ✨ 特性

- 🚀 **高性能**：优化的数据包处理和内存管理
- 🔄 **全兼容**：支持V2rayN、V2rayNG、Shadowrocket等所有主流客户端
- 🌐 **智能DNS**：使用1.1.1.1和8.8.8.8进行域名解析
- 📊 **性能监控**：实时内存使用和传输统计
- 🛡️ **内存安全**：自动清理和垃圾回收，长期运行稳定
- ⚡ **批处理**：智能数据包批处理，提升传输效率
- 🔧 **可配置**：丰富的环境变量配置选项
- 🎭 **主页伪装**：专业的云服务监控平台界面
- 🔒 **安全增强**：访问频率限制、攻击检测、订阅令牌验证
- 🚫 **防扫描**：自动检测和拦截可疑请求

## 环境变量配置

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `UUID` | `a2056d0d-c98e-4aeb-9aab-37f64edd5710` | UUID（强烈建议修改） |
| `AUTO_ACCESS` | `false` | 自动保活开关 |
| `XPATH` | UUID前8位 | XHTTP路径（建议自定义） |
| `SUB_PATH` | `sub` | 订阅路径（建议自定义） |
| `SUB_TOKEN` |  | 订阅访问令牌（可选，建议设置） |
| `DOMAIN` |   | 服务器域名或IP |
| `NAME` |     | 节点名称 |
| `PORT` | `3000` | HTTP服务端口 |
| `LOG_LEVEL` | `none` | 日志级别 (none/debug/info/warn/error) |
| `ENABLE_RATE_LIMIT` | `true` | 是否启用访问频率限制 |

### 订阅链接
* 无令牌: `https://your-domain.com/${SUB_PATH}`
* 有令牌: `https://your-domain.com/${SUB_PATH}?token=your-token`

> 💡 **安全提示**: 强烈建议设置 `SUB_TOKEN` 环境变量来保护订阅链接

### 使用cloudflare workers 或 snippets 反代域名给xhttp节点套cdn加速
```
export default {
    async fetch(request, env) {
        let url = new URL(request.url);
        if (url.pathname.startsWith('/')) {
            var arrStr = [
                'your-domain', // 此处单引号里填写你的节点伪装域名
            ];
            url.protocol = 'https:'
            url.hostname = getRandomArray(arrStr)
            let new_request = new Request(url, request);
            return fetch(new_request);
        }
        return env.ASSETS.fetch(request);
    },
};
function getRandomArray(array) {
  const randomIndex = Math.floor(Math.random() * array.length);
  return array[randomIndex];
}
```

## 快速部署 

### 1：源代码部署

#### 安装步骤

```bash
# 1. 克隆项目
git clone https://github.com/eooce/serverless-xhttp
cd serverless-xhttp

# 2. 安装依赖
npm install

# 3. 配置环境变量（可选）
export UUID=your-uuid-here
export DOMAIN=your-domain.com
export NAME=MyNode
export LOG_LEVEL=none

# 4. 启动服务
# 基本启动
node app.js

# 推荐启动（启用垃圾回收）
node --expose-gc app.js

# 生产环境启动（设置内存限制）
node --expose-gc --max-old-space-size=512 app.js
```

#### 使用PM2管理（推荐）

```bash
# 安装PM2
npm install -g pm2

# 创建PM2配置文件
cat > ecosystem.config.js << EOF
module.exports = {
  apps: [{
    name: 'xhttp',
    script: 'app.js',
    node_args: '--expose-gc --max-old-space-size=512',
    env: {
      UUID: 'your-uuid-here',
      DOMAIN: 'your-domain.com',
      NAME: 'MyNode',
      LOG_LEVEL: 'info'
    },
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G'
  }]
}
EOF

# 启动服务
pm2 start ecosystem.config.js

# 查看状态
pm2 status

# 查看日志
pm2 logs vless-xhttp
```

### 2：Docker部署

#### 使用预构建镜像

```bash
# 完整配置运行
docker run -d \
  --name vless-proxy \
  -p 3000:3000 \
  -e UUID=your-uuid-here \
  -e DOMAIN=your-domain.com \
  -e NAME=MyNode \
  --restart=unless-stopped \
  ghcr.io/eooce/xhttp:latest
```

#### 使用Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  xhttp:
    image: ghcr.io/eooce/xhttp:latest
    container_name: xhttp
    ports:
      - "3000:3000"
    environment:
      - NAME=MyNode
      - UUID=your-uuid-here
      - DOMAIN=your-domain.com
    restart: unless-stopped
```

```bash
# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 🔒 安全配置（重要）

### 基础安全配置
```bash
# 1. 修改默认UUID（必须）
export UUID="your-unique-uuid-here"

# 2. 自定义路径（强烈推荐）
export XPATH="x8k2m9p4"
export SUB_PATH="my-custom-path"

# 3. 启用订阅令牌（强烈推荐）
export SUB_TOKEN="your-secret-token"
```

### 完整安全配置示例
```bash
docker run -d \
  --name vless-proxy \
  -p 3000:3000 \
  -e UUID=your-unique-uuid \
  -e XPATH=custom-path-2024 \
  -e SUB_PATH=my-subscription \
  -e SUB_TOKEN=my-secret-token-123 \
  -e DOMAIN=your-domain.com \
  -e NAME=SecureNode \
  -e LOG_LEVEL=none \
  --restart=unless-stopped \
  ghcr.io/eooce/xhttp:latest
```

📖 **详细安全指南**: 请查看 [SECURITY.md](SECURITY.md) 了解完整的安全配置和最佳实践

### 温馨提示

* 如果使用的是IP:端口或域名:端口形式访问首页，请关闭节点的tls，并将节点端口改为运行的端口。
* 如果需要使用CDN功能，将IP解析到cloudflared，并设置端口回源，然后将节点的host和sni改为解析的域名。
* 请尽量确保在生产环境中使用HTTPS和有效的TLS证书。
* **生产环境必须修改默认UUID和路径，并启用订阅令牌验证！**

## 📄 开源协议

本项目采用 [MIT License](LICENSE) 开源协议。

---
## 如果此项目对你有帮助，请给一颗star

[![Star History Chart](https://api.star-history.com/svg?repos=eooce/serverless-xhttp&type=Date)](https://star-history.com/#eooce/serverless-xhttp&Date)
