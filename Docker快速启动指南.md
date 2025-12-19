# Open Notebook Docker 快速启动指南

## 📋 目录

- [前置要求](#前置要求)
- [快速开始](#快速开始)
- [详细配置](#详细配置)
- [界面使用指南](#界面使用指南)
- [常见问题](#常见问题)
- [停止和清理](#停止和清理)

---

## 前置要求

### 必需软件

1. **Docker** (版本 20.10+)
   ```bash
   docker --version
   ```

2. **Docker Compose** (版本 2.0+，推荐)
   ```bash
   docker compose version
   ```

### 必需信息

- **AI API密钥**：至少需要一个AI提供商的API密钥
  - OpenAI: `OPENAI_API_KEY`
  - Anthropic: `ANTHROPIC_API_KEY`
  - 或其他支持的提供商（见下方）

---

## 快速开始

### 方法一：Docker Compose（推荐）⭐

**优点**：配置简单，易于管理，推荐用于快速体验

#### 步骤 1：创建项目目录

```bash
mkdir open-notebook
cd open-notebook
```

#### 步骤 2：创建 docker-compose.yml

创建 `docker-compose.yml` 文件，内容如下：

```yaml
services:
  open_notebook:
    image: lfnovo/open_notebook:v1-latest-single
    # 或者使用 GitHub Container Registry:
    # image: ghcr.io/lfnovo/open-notebook:v1-latest-single
    
    ports:
      - "8502:8502"  # Web UI 前端界面
      - "5055:5055"  # REST API 后端（必需！）
    
    environment:
      # ===== AI 提供商配置 =====
      # 至少需要配置一个AI提供商的API密钥
      - OPENAI_API_KEY=your_openai_api_key_here
      
      # 如果需要使用其他提供商，取消注释并配置：
      # - ANTHROPIC_API_KEY=your_anthropic_key
      # - GROQ_API_KEY=your_groq_key
      # - GOOGLE_API_KEY=your_google_key
      
      # ===== 数据库配置（单容器模式）=====
      - SURREAL_URL=ws://localhost:8000/rpc
      - SURREAL_USER=root
      - SURREAL_PASSWORD=root
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=production
      
      # ===== 可选配置 =====
      # 如果从远程服务器访问，设置API_URL（替换为你的服务器IP）
      # - API_URL=http://192.168.1.100:5055
      
      # 可选：设置应用密码保护
      # - APP_PASSWORD=your_secure_password
    
    volumes:
      - ./notebook_data:/app/data          # 笔记本和研究内容
      - ./surreal_data:/mydata             # SurrealDB 数据库文件
    
    restart: always
```

#### 步骤 3：修改配置

**重要**：编辑 `docker-compose.yml`，将 `your_openai_api_key_here` 替换为你的实际API密钥。

#### 步骤 4：启动服务

```bash
docker compose up -d
```

#### 步骤 5：查看日志（可选）

```bash
docker compose logs -f
```

等待看到类似以下信息表示启动成功：
```
✅ Database is already at the latest version
✅ API initialization completed successfully
```

#### 步骤 6：访问应用

- **前端界面**：http://localhost:8502
- **API文档**：http://localhost:5055/docs
- **健康检查**：http://localhost:5055/health

---

### 方法二：Docker Run 命令

**适用场景**：本地机器快速测试，或需要更灵活的配置

#### 本地机器（同一台电脑访问）

```bash
mkdir open-notebook && cd open-notebook

docker run -d \
  --name open-notebook \
  -p 8502:8502 \
  -p 5055:5055 \
  -v ./notebook_data:/app/data \
  -v ./surreal_data:/mydata \
  -e OPENAI_API_KEY=your_openai_api_key_here \
  -e SURREAL_URL="ws://localhost:8000/rpc" \
  -e SURREAL_USER="root" \
  -e SURREAL_PASSWORD="root" \
  -e SURREAL_NAMESPACE="open_notebook" \
  -e SURREAL_DATABASE="production" \
  lfnovo/open_notebook:v1-latest-single
```

#### 远程服务器（从其他设备访问）

```bash
mkdir open-notebook && cd open-notebook

docker run -d \
  --name open-notebook \
  -p 8502:8502 \
  -p 5055:5055 \
  -v ./notebook_data:/app/data \
  -v ./surreal_data:/mydata \
  -e OPENAI_API_KEY=your_openai_api_key_here \
  -e API_URL=http://YOUR_SERVER_IP:5055 \
  -e SURREAL_URL="ws://localhost:8000/rpc" \
  -e SURREAL_USER="root" \
  -e SURREAL_PASSWORD="root" \
  -e SURREAL_NAMESPACE="open_notebook" \
  -e SURREAL_DATABASE="production" \
  lfnovo/open_notebook:v1-latest-single
```

**注意**：将 `YOUR_SERVER_IP` 替换为你的服务器实际IP地址（如 `192.168.1.100`）

---

## 详细配置

### 端口说明

| 端口 | 用途 | 必需性 |
|------|------|--------|
| 8502 | Next.js 前端界面 | ✅ 必需 |
| 5055 | FastAPI 后端API | ✅ 必需 |
| 8000 | SurrealDB（容器内部） | 自动配置 |

### 环境变量配置

#### AI 提供商配置

**OpenAI**
```yaml
- OPENAI_API_KEY=sk-...
```

**Anthropic**
```yaml
- ANTHROPIC_API_KEY=sk-ant-...
```

**Groq**
```yaml
- GROQ_API_KEY=gsk_...
```

**Google (GenAI)**
```yaml
- GOOGLE_API_KEY=...
```

**Ollama（本地运行）**
```yaml
- OLLAMA_BASE_URL=http://host.docker.internal:11434
```

**Azure OpenAI**
```yaml
- AZURE_OPENAI_API_KEY=...
- AZURE_OPENAI_ENDPOINT=https://...
- AZURE_OPENAI_DEPLOYMENT_NAME=...
```

#### 数据库配置

单容器模式下，数据库配置通常是固定的：
```yaml
- SURREAL_URL=ws://localhost:8000/rpc
- SURREAL_USER=root
- SURREAL_PASSWORD=root
- SURREAL_NAMESPACE=open_notebook
- SURREAL_DATABASE=production
```

#### 安全配置

**应用密码保护**（可选）
```yaml
- APP_PASSWORD=your_secure_password
```

设置后，访问应用时需要输入密码。

#### 远程访问配置

**API_URL**（从其他设备访问时必需）
```yaml
# 使用IP地址
- API_URL=http://192.168.1.100:5055

# 或使用域名
- API_URL=http://myserver.local:5055
```

**重要提示**：
- ✅ 使用 `http://192.168.1.100:5055` 格式
- ❌ 不要使用 `localhost`（远程访问时无效）
- ❌ 不要添加 `/api` 后缀

---

## 界面使用指南

### 首次启动后的界面

启动成功后，访问 http://localhost:8502，你会看到：

```
┌─────────────────────────────────────────────────────────┐
│  Open Notebook                                          │
├──────────┬──────────┬──────────────────────────────────┤
│          │          │                                   │
│ Sources  │  Notes   │  Chat                            │
│ (源文件) │  (笔记)  │  (聊天)                          │
│          │          │                                   │
│          │          │                                   │
└──────────┴──────────┴──────────────────────────────────┘
```

### 三栏界面说明

#### 1. 左侧栏：Sources（源文件）

**功能**：
- 上传和管理研究材料
- 支持多种格式：PDF、视频、音频、网页、Office文档等

**操作步骤**：
1. 点击 "Add Source" 按钮
2. 选择源文件类型（文件上传、URL、文本等）
3. 等待处理完成（自动提取内容和生成嵌入向量）

#### 2. 中间栏：Notes（笔记）

**功能**：
- 创建和管理笔记
- 支持手动创建和AI生成

**操作步骤**：
1. 点击 "Add Note" 按钮
2. 选择创建方式：
   - **Manual Note**：手动编写
   - **AI Note**：基于源文件AI生成

#### 3. 右侧栏：Chat（聊天）

**功能**：
- 基于研究内容的AI对话
- 多会话管理
- 引用来源显示

**操作步骤**：
1. 在聊天框中输入问题
2. AI会基于已添加的源文件内容回答
3. 查看引用来源（点击引用标记）

### 快速上手指南

#### 步骤 1：创建笔记本

1. 点击侧边栏的 "New Notebook"
2. 输入笔记本名称和描述
3. 点击创建

#### 步骤 2：添加源文件

1. 在 Sources 栏点击 "Add Source"
2. 选择 "Upload File"
3. 上传一个PDF文件（或其他格式）
4. 等待处理完成（会显示处理进度）

#### 步骤 3：开始对话

1. 切换到 Chat 栏
2. 输入问题，例如：
   - "总结一下这个文档的主要内容"
   - "文档中提到了哪些关键概念？"
   - "这个文档的核心观点是什么？"
3. AI会基于上传的文档内容回答

#### 步骤 4：搜索内容

1. 使用顶部的搜索框
2. 输入关键词
3. 查看搜索结果（全文搜索 + 向量搜索）

#### 步骤 5：创建笔记

1. 切换到 Notes 栏
2. 点击 "Add Note"
3. 选择 "AI Note"
4. 选择要基于的源文件
5. AI会自动生成笔记

---

## 常见问题

### 1. 无法访问界面（空白页面）

**可能原因**：
- 端口未正确暴露
- API_URL配置错误（远程访问时）

**解决方法**：
```bash
# 检查端口是否暴露
docker ps | grep open-notebook

# 检查日志
docker compose logs

# 确保两个端口都已映射
# -p 8502:8502 -p 5055:5055
```

### 2. "Unable to connect to server" 错误

**可能原因**：
- API_URL未设置或设置错误（远程访问时）

**解决方法**：
```yaml
# 在 docker-compose.yml 中添加
- API_URL=http://YOUR_SERVER_IP:5055
```

**注意**：
- ✅ 使用实际IP：`http://192.168.1.100:5055`
- ❌ 不要使用：`http://localhost:5055`（远程访问时）

### 3. API密钥错误

**错误信息**：
- "Invalid API key"
- "Authentication failed"

**解决方法**：
1. 检查环境变量是否正确设置
2. 确认API密钥有效
3. 查看日志确认密钥格式：
```bash
docker compose logs | grep -i "api\|key\|auth"
```

### 4. 容器启动失败

**检查步骤**：
```bash
# 查看容器状态
docker ps -a | grep open-notebook

# 查看详细日志
docker compose logs --tail=100

# 检查端口占用
netstat -tulpn | grep -E "8502|5055"
```

### 5. 数据库连接错误

**错误信息**：
- "Failed to connect to database"
- "Database migration failed"

**解决方法**：
1. 检查数据库配置环境变量
2. 确保SurrealDB在容器内正常运行
3. 查看数据库日志：
```bash
docker compose logs | grep -i "surreal\|database"
```

### 6. 文件上传失败

**可能原因**：
- 文件太大
- 文件格式不支持
- 磁盘空间不足

**解决方法**：
```bash
# 检查磁盘空间
df -h

# 检查数据目录权限
ls -la notebook_data/
```

### 7. 处理速度慢

**优化建议**：
1. 使用更快的AI提供商（如Groq）
2. 减少源文件大小
3. 检查网络连接（如果使用云API）

---

## 停止和清理

### 停止服务

**使用 Docker Compose**：
```bash
docker compose down
```

**使用 Docker Run**：
```bash
docker stop open-notebook
docker rm open-notebook
```

### 停止但保留数据

**使用 Docker Compose**：
```bash
docker compose stop
```

**使用 Docker Run**：
```bash
docker stop open-notebook
```

### 完全清理（删除所有数据）

**警告**：这会删除所有笔记本和数据！

```bash
# 停止并删除容器
docker compose down

# 删除数据目录
rm -rf notebook_data surreal_data
```

### 备份数据

```bash
# 备份数据目录
tar -czf open-notebook-backup-$(date +%Y%m%d).tar.gz \
  notebook_data/ surreal_data/

# 恢复备份
tar -xzf open-notebook-backup-YYYYMMDD.tar.gz
```

---

## 进阶配置

### 使用环境变量文件

创建 `.env` 文件：

```bash
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
APP_PASSWORD=your_password
API_URL=http://192.168.1.100:5055
```

在 `docker-compose.yml` 中引用：

```yaml
services:
  open_notebook:
    env_file:
      - .env
    # ... 其他配置
```

### 自定义端口

如果8502或5055端口被占用，可以修改：

```yaml
ports:
  - "8080:8502"  # 前端改为8080
  - "9090:5055"  # API改为9090
```

然后访问：http://localhost:8080

### 使用反向代理

如果使用Nginx等反向代理，只需代理到8502端口：

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8502;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 支持的AI提供商

| 提供商 | LLM | Embedding | STT | TTS | 配置变量 |
|--------|-----|-----------|-----|-----|----------|
| OpenAI | ✅ | ✅ | ✅ | ✅ | `OPENAI_API_KEY` |
| Anthropic | ✅ | ❌ | ❌ | ❌ | `ANTHROPIC_API_KEY` |
| Groq | ✅ | ❌ | ✅ | ❌ | `GROQ_API_KEY` |
| Google (GenAI) | ✅ | ✅ | ❌ | ✅ | `GOOGLE_API_KEY` |
| Ollama | ✅ | ✅ | ❌ | ❌ | `OLLAMA_BASE_URL` |
| Azure OpenAI | ✅ | ✅ | ❌ | ❌ | `AZURE_OPENAI_*` |
| Mistral | ✅ | ✅ | ❌ | ❌ | `MISTRAL_API_KEY` |
| DeepSeek | ✅ | ❌ | ❌ | ❌ | `DEEPSEEK_API_KEY` |

更多提供商配置请参考：[AI Models Documentation](docs/features/ai-models.md)

---

## 获取帮助

### 官方资源

- 📖 [完整文档](docs/getting-started/index.md)
- 💬 [Discord社区](https://discord.gg/37XJPXfz2w)
- 🐛 [GitHub Issues](https://github.com/lfnovo/open-notebook/issues)
- 🌐 [官方网站](https://www.open-notebook.ai)

### 日志查看

```bash
# 实时查看日志
docker compose logs -f

# 查看最近100行
docker compose logs --tail=100

# 查看特定服务日志
docker compose logs open_notebook
```

---

## 总结

通过以上步骤，你应该能够：

1. ✅ 使用Docker快速启动Open Notebook
2. ✅ 了解基本配置选项
3. ✅ 熟悉界面操作
4. ✅ 解决常见问题
5. ✅ 进行数据备份和恢复

**下一步**：
- 阅读 [用户指南](docs/user-guide/index.md) 了解更多功能
- 查看 [API文档](http://localhost:5055/docs) 了解API使用
- 探索 [高级功能](docs/features/index.md)

祝你使用愉快！🎉

---

*最后更新：2024年*
*文档版本：1.0*
