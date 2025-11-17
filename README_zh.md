# Skim MCP Server

> 🚀 生产就绪的 Model Context Protocol 服务器，用于 Skim 代码转换

[![版本](https://img.shields.io/npm/v/skim-mcp-server?style=flat-square)](https://www.npmjs.com/package/skim-mcp-server)
[![许可证](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen?style=flat-square)](https://nodejs.org/)

智能地为 LLM 上下文窗口压缩代码，内置安全性、监控和生产级功能。

## 🌟 特性

- 🔒 **默认安全** - 路径验证、输入清理、速率限制
- 📊 **生产监控** - 结构化日志、健康检查、指标
- 🚀 **高性能** - 使用 spawn 快速执行，优化的缓冲
- 🛡️ **DoS 保护** - 请求限制、大小限制、超时处理
- 📝 **全面日志** - Winston 集成，可配置日志级别
- 🧪 **完整测试覆盖** - 包含单元和集成测试
- 📦 **零依赖** - 仅 MCP SDK 和 Winston（生产依赖）

## 📦 安装

### 前置要求

- Node.js >= 18.0.0
- npm、pnpm 或 yarn

### 选项 1: 全局安装（推荐）

```bash
# 安装 MCP 服务器和 Skim CLI
npm install -g skim-mcp-server

# 或单独安装 skim CLI
npm install -g rskim
```

### 选项 2: 项目安装

```bash
# 在你的项目目录中
npm install skim-mcp-server
```

### 选项 3: 从源码安装

```bash
git clone https://github.com/luw2007/skim-mcp-server.git
cd skim-mcp-server
npm install
npm run build
```

### 自动安装 Skim

如果未找到，服务器会自动安装 Skim CLI：

```bash
# 在 npm install 期间（postinstall 钩子）
npm install skim-mcp-server

# 或手动安装
npm run install-skim
```

## 🔧 配置

### Claude Code 设置

添加到你的 Claude Code 配置（通常是 `~/.config/claude-code/config.json`）：

```json
{
  "mcpServers": {
    "skim": {
      "command": "skim-mcp-server"
    }
  }
}
```

### 环境变量

```bash
# 日志级别
export LOG_LEVEL=info  # debug、info、warn、error

# 允许的基准路径（逗号分隔）
export SKIM_ALLOWED_PATHS=/workspace,/home/user/projects

# 速率限制
export SKIM_MAX_REQUESTS_PER_MINUTE=10

# 输入大小限制
export SKIM_MAX_INPUT_SIZE=10485760  # 10MB，以字节为单位
```

## 🚀 使用

配置完成后，工具会自动在 Claude Code 中可用：

### 工具 1: `skim_transform` - 转换源代码

从字符串转换代码：

```javascript
// Claude Code 在分析代码时自动使用此工具
mcp__skim__skim_transform({
  source: 'function add(a, b) { return a + b; }',
  language: 'javascript',
  mode: 'structure',
  show_stats: true
})

// 返回：
// function add(a, b)  { /* ... */ }
//
// 📊 Token 减少统计：
// [skim] 24 tokens → 9 tokens (62.5% 减少)
```

### 工具 2: `skim_file` - 转换文件

转换文件或目录：

```javascript
mcp__skim__skim_file({
  path: '/workspace/src',
  mode: 'structure',
  show_stats: true
})

// 返回压缩后的代码和统计信息
```

### 工具 3: `skim_analyze` - 架构分析

分析代码架构：

```javascript
mcp__skim__skim_analyze({
  path: '/workspace/src',
  mode: 'structure'
})

// 返回：
// 1. 压缩后的代码
// 2. Token 统计
// 3. 分析框架来指导 Claude
```

### 自然使用

Claude Code 在适当的时候自动使用这些工具：

```
用户: "分析 src/ 的架构"
 → Claude 自动调用 skim_analyze
   → 接收压缩后的代码（缩小 60%）
     → 提供包含完整上下文的更好分析

用户: "审查这个 TypeScript 函数"
 → Claude 自动调用 skim_transform
   → 获得干净的方法签名
     → 提供专注的审查
```

## 🔒 安全特性

### 路径验证

✅ 仅允许绝对路径
✅ 防止路径遍历攻击（`../../../etc/passwd`）
✅ 带有验证的符号链接解析
✅ 可配置的允许基准路径

### 输入清理

✅ 最大输入大小（默认 10MB）
✅ 空字节检测
✅ 命令注入预防
✅ Shell 注入缓解（参数化命令）

### 速率限制

✅ 每分钟请求数限制
✅ 防止 DoS 攻击
✅ Retry-after 头部

## 🧪 测试

### 运行测试

```bash
# 安装依赖
npm install

# 运行所有测试
npm test

# 带覆盖率测试
npm run test:coverage

# 开发监听模式
npm run test:watch
```

### 测试覆盖

- ✅ Skim CLI 可用性
- ✅ 路径遍历预防
- ✅ 输入验证
- ✅ 超大输入检测
- ✅ 无效语言检测
- ✅ 空字节拒绝
- ✅ 畸形路径处理

## 📊 性能

### 基准测试

```bash
# 单个文件（300 行）
skim transform - 1.3ms

# 大文件（3000 行）
skim transform - 14.6ms

# 缓存（第二次运行）
skim transform - 5ms（快 48 倍）

# MCP 开销 < 2ms
```

### 资源限制

- 最大输入：每个请求 10MB
- 最大输出：50MB 缓冲区
- 超时：每个请求 30 秒
- 速率限制：每分钟 10 个请求

## 📖 文档

### 转换模式

| 模式 | 减少率 | 使用场景 | 示例输出 |
|------|--------|----------|----------|
| **structure** | 60-80% | 架构分析 | `function foo() { /* ... */ }` |
| **signatures** | 85-92% | API 文档 | `function foo(): void` |
| **types** | 90-95% | 类型系统分析 | `interface User { name: string }` |
| **full** | 0% | 调试/验证 | 原始代码 |

### 支持的语言

- TypeScript / JavaScript
- Python
- Rust
- Go
- Java
- JSON（特殊结构模式）
- Markdown（头部提取）

### CLI 参考

```bash
# 转换文件
skim file.ts --mode=structure --show-stats

# 转换目录
skim src/ --mode=signatures

# 使用 glob 转换
skim 'src/**/*.ts' --jobs 4

# 清除缓存
skim --clear-cache
```

## 🛠️ 开发

### 设置

```bash
git clone https://github.com/luw2007/skim-mcp-server.git
cd skim-mcp-server
npm install
```

### 开发工作流

```bash
# 开始开发
npm run dev

# 检查代码
npm run lint

# 修复检查问题
npm run lint:fix

# 格式化代码
npm run format:fix

# 构建生产版本
npm run build
```

### 项目结构

```
skim-mcp-server/
├── src/
│   └── index.js          # 主服务器
├── test/
│   └── index.test.js     # 测试套件
├── scripts/
│   ├── install-skim.js   # 自动安装脚本
│   └── build.js          # 构建脚本
├── docs/
│   └── examples.md       # 使用示例
├── dist/                 # 构建文件
└── README.md
```

## 🐳 Docker

### 构建镜像

```bash
docker build -t skim-mcp-server .
```

### 运行容器

```bash
docker run -i --rm \
  -e LOG_LEVEL=info \
  -v /workspace:/workspace \
  skim-mcp-server
```

### Docker Compose

```yaml
version: '3.8'
services:
  skim-mcp:
    image: skim-mcp-server
    environment:
      - LOG_LEVEL=info
      - SKIM_ALLOWED_PATHS=/workspace
    volumes:
      - ./workspace:/workspace
```

## 📄 许可证

MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🤝 贡献

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

### 开发设置

1. Fork 仓库
2. 创建功能分支
3. 做出更改
4. 添加测试
5. 确保检查通过
6. 提交 pull request

## 🆘 支持

### 报告问题

请在 [GitHub Issues](https://github.com/luw2007/skim-mcp-server/issues) 报告问题。

包含信息：
- Node.js 版本 (`node --version`)
- 操作系统和架构
- 重现步骤
- 错误日志

### 获取帮助

- 📖 文档: [docs/](docs/)
- 💡 示例: [docs/examples.md](docs/examples.md)
- 💬 讨论: [GitHub Discussions](https://github.com/luw2007/skim-mcp-server/discussions)

## 🔄 更新日志

查看 [CHANGELOG.md](CHANGELOG.md) 了解版本历史

## 🔮 路线图

### v1.1.0 (下一个版本)

- [ ] HTTP 传输支持
- [ ] WebSocket 传输
- [ ] 插件系统
- [ ] 自定义转换规则
- [ ] 集成更多 LLM 平台

### v1.2.0 (未来)

- [ ] 并行处理优化
- [ ] 内存高效流式处理
- [ ] 高级缓存策略
- [ ] 指标仪表板

## 🙏 致谢

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [tree-sitter](https://tree-sitter.github.io/)
- [Skim](https://github.com/dean0x/skim) - dean0x 的上游 Skim 项目
- Claude Code 团队

---

**为 LLM 社区打造 ❤️**
