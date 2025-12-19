# OpenAI Base URL 覆盖配置指南

## 📋 概述

Open Notebook 支持覆盖 OpenAI API 的 Base URL，允许你使用：
- 🏠 **本地代理服务器**（如 LM Studio、vLLM）
- 🌐 **自定义 OpenAI 兼容端点**
- 🔒 **企业代理或网关**
- 💰 **第三方 OpenAI 兼容服务**

---

## ✅ 是的，你可以覆盖 OpenAI Base URL！

Open Notebook 通过 **`OPENAI_COMPATIBLE_BASE_URL`** 环境变量来覆盖 OpenAI 的默认 Base URL。

---

## 🚀 快速配置

### 方法一：覆盖所有 OpenAI 服务（最简单）

**适用于**：所有服务（LLM、Embedding、STT、TTS）使用同一个端点

```bash
# 设置环境变量
export OPENAI_COMPATIBLE_BASE_URL=http://your-custom-endpoint.com/v1

# 如果端点需要API密钥
export OPENAI_COMPATIBLE_API_KEY=your_api_key_here
```

**Docker Compose 配置示例**：
```yaml
services:
  open_notebook:
    image: lfnovo/open_notebook:v1-latest-single
    environment:
      - OPENAI_COMPATIBLE_BASE_URL=http://your-custom-endpoint.com/v1
      - OPENAI_COMPATIBLE_API_KEY=your_api_key_here
      # 其他配置...
```

**Docker Run 配置示例**：
```bash
docker run -d \
  --name open-notebook \
  -p 8502:8502 -p 5055:5055 \
  -e OPENAI_COMPATIBLE_BASE_URL=http://your-custom-endpoint.com/v1 \
  -e OPENAI_COMPATIBLE_API_KEY=your_api_key_here \
  -v ./notebook_data:/app/data \
  -v ./surreal_data:/mydata \
  lfnovo/open_notebook:v1-latest-single
```

---

### 方法二：按服务类型分别配置（高级）

**适用于**：不同服务使用不同的端点

```bash
# 语言模型（LLM）
export OPENAI_COMPATIBLE_BASE_URL_LLM=http://localhost:1234/v1

# 嵌入向量（Embedding）
export OPENAI_COMPATIBLE_BASE_URL_EMBEDDING=http://localhost:8080/v1

# 语音转文字（STT）
export OPENAI_COMPATIBLE_BASE_URL_STT=http://localhost:9000/v1

# 文字转语音（TTS）
export OPENAI_COMPATIBLE_BASE_URL_TTS=http://localhost:8969/v1
```

**Docker Compose 配置示例**：
```yaml
services:
  open_notebook:
    image: lfnovo/open_notebook:v1-latest-single
    environment:
      # 按服务类型配置
      - OPENAI_COMPATIBLE_BASE_URL_LLM=http://localhost:1234/v1
      - OPENAI_COMPATIBLE_BASE_URL_EMBEDDING=http://localhost:8080/v1
      - OPENAI_COMPATIBLE_BASE_URL_STT=http://localhost:9000/v1
      - OPENAI_COMPATIBLE_BASE_URL_TTS=http://localhost:8969/v1
      
      # 如果需要API密钥
      - OPENAI_COMPATIBLE_API_KEY_LLM=llm_key_here
      - OPENAI_COMPATIBLE_API_KEY_EMBEDDING=embedding_key_here
      # 其他配置...
```

---

## 📝 环境变量说明

### 通用配置变量

| 变量名 | 用途 | 覆盖范围 |
|--------|------|----------|
| `OPENAI_COMPATIBLE_BASE_URL` | 所有AI服务的Base URL | LLM、Embedding、STT、TTS |
| `OPENAI_COMPATIBLE_API_KEY` | 所有服务的API密钥 | LLM、Embedding、STT、TTS |

### 按服务类型配置变量

| 变量名 | 用途 | 优先级 |
|--------|------|--------|
| `OPENAI_COMPATIBLE_BASE_URL_LLM` | 语言模型端点 | ⬆️ 最高 |
| `OPENAI_COMPATIBLE_BASE_URL_EMBEDDING` | 嵌入向量端点 | ⬆️ 最高 |
| `OPENAI_COMPATIBLE_BASE_URL_STT` | 语音转文字端点 | ⬆️ 最高 |
| `OPENAI_COMPATIBLE_BASE_URL_TTS` | 文字转语音端点 | ⬆️ 最高 |
| `OPENAI_COMPATIBLE_API_KEY_LLM` | LLM API密钥 | - |
| `OPENAI_COMPATIBLE_API_KEY_EMBEDDING` | Embedding API密钥 | - |
| `OPENAI_COMPATIBLE_API_KEY_STT` | STT API密钥 | - |
| `OPENAI_COMPATIBLE_API_KEY_TTS` | TTS API密钥 | - |

**优先级规则**：
- 服务特定的变量（如 `OPENAI_COMPATIBLE_BASE_URL_LLM`）会覆盖通用变量（`OPENAI_COMPATIBLE_BASE_URL`）
- 如果设置了 `OPENAI_COMPATIBLE_BASE_URL_LLM`，LLM服务会使用这个URL，而不是 `OPENAI_COMPATIBLE_BASE_URL`

---

## 🎯 常见使用场景

### 场景 1：使用 LM Studio（本地模型）

**配置**：
```bash
export OPENAI_COMPATIBLE_BASE_URL=http://localhost:1234/v1
```

**说明**：
- LM Studio 通常不需要API密钥
- 确保LM Studio的本地服务器已启动
- 默认端口是1234

**完整Docker Compose示例**：
```yaml
services:
  open_notebook:
    image: lfnovo/open_notebook:v1-latest-single
    ports:
      - "8502:8502"
      - "5055:5055"
    environment:
      - OPENAI_COMPATIBLE_BASE_URL=http://host.docker.internal:1234/v1
      - SURREAL_URL=ws://localhost:8000/rpc
      - SURREAL_USER=root
      - SURREAL_PASSWORD=root
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=production
    volumes:
      - ./notebook_data:/app/data
      - ./surreal_data:/mydata
```

---

### 场景 2：使用企业代理或网关

**配置**：
```bash
export OPENAI_COMPATIBLE_BASE_URL=https://your-proxy.company.com/v1
export OPENAI_COMPATIBLE_API_KEY=your_proxy_api_key
```

**说明**：
- 使用HTTPS确保安全
- 代理服务器需要实现OpenAI兼容的API
- 可能需要自定义SSL证书配置

---

### 场景 3：使用第三方 OpenAI 兼容服务

**配置**：
```bash
export OPENAI_COMPATIBLE_BASE_URL=https://api.third-party-service.com/v1
export OPENAI_COMPATIBLE_API_KEY=your_third_party_key
```

**示例服务**：
- Together AI
- Anyscale Endpoints
- Replicate
- LocalAI

---

### 场景 4：混合配置（本地+云端）

**配置**：
```bash
# 本地LLM（隐私敏感）
export OPENAI_COMPATIBLE_BASE_URL_LLM=http://localhost:1234/v1

# 云端Embedding（更好的质量）
export OPENAI_API_KEY=your_openai_key  # 使用官方OpenAI的Embedding
```

**说明**：
- LLM使用本地端点（隐私保护）
- Embedding使用官方OpenAI（更好的质量）
- 其他服务可以单独配置

---

### 场景 5：Docker容器访问宿主机服务

**macOS/Windows**：
```bash
export OPENAI_COMPATIBLE_BASE_URL=http://host.docker.internal:1234/v1
```

**Linux**：
```bash
# 方法1：使用Docker网关IP
export OPENAI_COMPATIBLE_BASE_URL=http://172.17.0.1:1234/v1

# 方法2：使用host网络模式
docker run --network host ...
```

---

## 🔧 详细配置示例

### 完整 Docker Compose 配置

```yaml
services:
  open_notebook:
    image: lfnovo/open_notebook:v1-latest-single
    ports:
      - "8502:8502"  # Web UI
      - "5055:5055"  # API
    environment:
      # ===== OpenAI兼容端点配置 =====
      # 方式1：统一配置（所有服务使用同一端点）
      - OPENAI_COMPATIBLE_BASE_URL=http://localhost:1234/v1
      - OPENAI_COMPATIBLE_API_KEY=not_needed  # LM Studio通常不需要
      
      # 方式2：分别配置（不同服务使用不同端点）
      # - OPENAI_COMPATIBLE_BASE_URL_LLM=http://localhost:1234/v1
      # - OPENAI_COMPATIBLE_BASE_URL_EMBEDDING=http://localhost:8080/v1
      # - OPENAI_COMPATIBLE_BASE_URL_STT=http://localhost:9000/v1
      # - OPENAI_COMPATIBLE_BASE_URL_TTS=http://localhost:8969/v1
      
      # ===== 数据库配置 =====
      - SURREAL_URL=ws://localhost:8000/rpc
      - SURREAL_USER=root
      - SURREAL_PASSWORD=root
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=production
      
      # ===== 其他可选配置 =====
      # - API_URL=http://192.168.1.100:5055  # 远程访问时设置
      # - APP_PASSWORD=your_password  # 应用密码保护
    volumes:
      - ./notebook_data:/app/data
      - ./surreal_data:/mydata
    restart: always
```

---

## 🔍 验证配置

### 1. 检查环境变量

```bash
# 在容器内检查
docker exec open-notebook env | grep OPENAI_COMPATIBLE
```

### 2. 测试端点连接

```bash
# 测试端点是否可访问
curl http://localhost:1234/v1/models

# 如果使用API密钥
curl -H "Authorization: Bearer YOUR_KEY" \
  http://your-endpoint.com/v1/models
```

### 3. 查看Open Notebook日志

```bash
# 查看容器日志
docker logs open-notebook | grep -i "openai\|compatible\|base"

# 实时查看日志
docker logs -f open-notebook
```

### 4. 在Open Notebook中测试

1. 访问 http://localhost:8502
2. 进入 Settings（设置）
3. 查看 Models（模型）配置
4. 尝试使用配置的模型进行对话

---

## ⚠️ 注意事项

### 1. URL格式要求

✅ **正确格式**：
```
http://localhost:1234/v1
https://api.example.com/v1
http://192.168.1.100:1234/v1
```

❌ **错误格式**：
```
http://localhost:1234          # 缺少 /v1
http://localhost:1234/         # 末尾斜杠（可能有问题）
localhost:1234/v1              # 缺少协议
```

### 2. Docker网络配置

**问题**：容器内无法访问 `localhost`

**解决方案**：
- **macOS/Windows**：使用 `host.docker.internal`
- **Linux**：使用 `172.17.0.1` 或 `--network host`

### 3. SSL证书验证

如果使用自签名证书或HTTPS代理，可能需要配置：

```bash
# 方法1：使用自定义CA证书
export ESPERANTO_SSL_CA_BUNDLE=/path/to/ca-bundle.pem

# 方法2：禁用SSL验证（仅开发环境）
export ESPERANTO_SSL_VERIFY=false
```

### 4. API密钥配置

- 如果端点不需要API密钥，可以设置为任意值或留空
- 某些服务（如LM Studio）不需要API密钥
- 确保API密钥格式正确（Bearer token格式）

---

## 🐛 故障排除

### 问题1：连接被拒绝

**症状**：`Connection refused` 或 `Could not connect`

**解决方案**：
1. 确认端点服务正在运行
2. 检查端口是否正确
3. 验证URL格式（必须包含 `/v1`）
4. 对于Docker，使用 `host.docker.internal` 而不是 `localhost`

### 问题2：模型未找到

**症状**：`Model not found` 或 `No models available`

**解决方案**：
1. 确认端点支持OpenAI兼容API
2. 检查端点是否加载了模型
3. 测试端点：`curl http://your-endpoint/v1/models`
4. 查看端点文档确认模型名称

### 问题3：认证失败

**症状**：`Unauthorized` 或 `Invalid API key`

**解决方案**：
1. 检查API密钥是否正确设置
2. 验证端点是否需要API密钥
3. 测试API密钥：`curl -H "Authorization: Bearer KEY" http://endpoint/v1/models`
4. 检查环境变量是否正确传递到容器

### 问题4：SSL证书错误

**症状**：`SSL: CERTIFICATE_VERIFY_FAILED`

**解决方案**：
```bash
# 使用自定义CA证书
export ESPERANTO_SSL_CA_BUNDLE=/path/to/ca-bundle.pem

# 或禁用验证（仅开发环境）
export ESPERANTO_SSL_VERIFY=false
```

---

## 📚 相关文档

- [OpenAI兼容端点完整指南](docs/features/openai-compatible.md)
- [AI模型配置指南](docs/features/ai-models.md)
- [本地TTS设置](docs/features/local_tts.md)
- [Docker快速启动指南](Docker快速启动指南.md)

---

## 💡 最佳实践

### 1. 使用环境变量文件

创建 `.env` 文件：
```bash
OPENAI_COMPATIBLE_BASE_URL=http://localhost:1234/v1
OPENAI_COMPATIBLE_API_KEY=not_needed
```

在 `docker-compose.yml` 中引用：
```yaml
services:
  open_notebook:
    env_file:
      - .env
```

### 2. 安全配置

- ✅ 使用环境变量，不要硬编码密钥
- ✅ 生产环境使用HTTPS
- ✅ 定期轮换API密钥
- ✅ 使用防火墙限制访问

### 3. 性能优化

- 本地端点：使用 `localhost` 或 `127.0.0.1`（最低延迟）
- 远程端点：选择地理位置近的服务器
- 监控连接：定期检查端点健康状态

---

## ✅ 总结

**是的，你可以覆盖 OpenAI Base URL！**

通过设置 `OPENAI_COMPATIBLE_BASE_URL` 环境变量，你可以：
- ✅ 使用本地模型（LM Studio、vLLM等）
- ✅ 使用企业代理或网关
- ✅ 使用第三方OpenAI兼容服务
- ✅ 按服务类型分别配置端点

**快速开始**：
```bash
export OPENAI_COMPATIBLE_BASE_URL=http://your-endpoint.com/v1
```

**在Docker中**：
```yaml
environment:
  - OPENAI_COMPATIBLE_BASE_URL=http://your-endpoint.com/v1
```

---

*最后更新：2024年*
*文档版本：1.0*
