# OBC Coin API

一个用于创建和发布代币的 API 服务，支持代币模板生成、编译和发布到 Benfen 网络。

## 功能特性

- 🪙 代币模板生成和自定义
- 🔧 自动编译 Move 智能合约
- 🚀 一键发布到 Benfen 网络
- 📝 清晰的参数结构和错误处理

## API 接口

### 1. 添加代币 - `/api/token/add`

创建代币模板并编译生成字节码。

**请求方法：** `POST`

**请求参数：**
```json
{
  "decimal": 8,
  "symbol": "TEST",
  "name": "Test Token",
  "description": "A test token for complete workflow",
  "json": {
    "website": "https://test.com",
    "twitter": "@test"
  }
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "代币添加和编译成功",
  "data": {
    "compile_output": "编译输出信息...",
    "output_file": "/path/to/generated/file.move",
    "request": {...}
  }
}
```

### 2. 发布代币 - `/api/token/publish`

将编译后的代币发布到 Benfen 网络。

**请求方法：** `POST`

**请求参数：**
```json
{
  "sender": "发送者地址",
  "compiled_modules": ["编译后的模块数组"],
  "dependencies": ["依赖项数组"],
  "gas_budget": "Gas 预算"
}
```

## 完整测试流程

### 步骤 1：创建代币

```bash
curl -X POST http://localhost:8080/api/token/add \
  -H "Content-Type: application/json" \
  -d '{
    "decimal": 8,
    "symbol": "TEST",
    "name": "Test Token",
    "description": "A test token for complete workflow",
    "json": {
      "website": "https://test.com",
      "twitter": "@test"
    }
  }'
```

**预期响应：**
- 状态码：200
- 包含编译输出、模块和依赖信息
- 响应时间：约 2-3 秒

### 步骤 2：发布代币

使用步骤 1 返回的编译结果：

```bash
curl -X POST http://localhost:8080/api/token/publish \
  -H "Content-Type: application/json" \
  -d '{
    "sender": "BFC6f0f9a9a72f7d48b8fcbfa09ebb61123d847aaad5760297d68c64795bad514b14a89",
    "compiled_modules": ["从步骤1获取的编译模块"],
    "dependencies": ["从步骤1获取的依赖项"],
    "gas_budget": "5000000000"
  }'
```

**预期响应：**
- 状态码：200
- 来自 Benfen RPC 的发布结果
- 响应时间：约 300-500 毫秒

## 配置说明

服务器配置文件 `config.yaml`：

```yaml
bfc:
  directory: "/path/to/bfc"
  binary_path: "/path/to/bfc/target/debug/bfc"
  token_template_path: "/path/to/token/template"

benfen_rpc:
  url: "https://devrpc2.benfen.org/"
  timeout: 30
  retry_count: 3

server:
  port: 8080
```

## 启动服务

### 方式一：使用一键脚本（推荐）

```bash
# 克隆项目
git clone <repository-url>
cd obc_coin_api

# 安装依赖
go mod tidy

# 启动服务
./start.sh

# 查看状态
./status.sh

# 停止服务
./stop.sh

# 重启服务
./restart.sh
```

### 方式二：手动启动

```bash
# 直接运行
go run .
```

服务启动后将在 `localhost:8080` 监听请求。

## 服务管理脚本

项目提供了完整的服务管理脚本：

- `./start.sh` - 启动服务（后台运行）
- `./stop.sh` - 停止服务
- `./restart.sh` - 重启服务
- `./status.sh` - 查看服务状态

**脚本特性：**
- 🔄 自动进程管理和 PID 文件处理
- 📝 日志文件自动生成（`obc_coin_api.log`）
- 🏥 健康检查和状态监控
- 🛡️ 安全的启停流程
- 🧹 自动清理过期的临时目录

## 自动清理功能

服务启动时会自动启动定时清理任务：

- **清理频率**: 可在配置文件中设置执行间隔
- **清理规则**: 可在配置文件中设置目录保留时间
- **清理范围**: 只清理 `coin_tmp_时间戳` 格式的目录
- **日志记录**: 详细记录清理过程和结果
- **配置化**: 支持通过 `config.yaml` 自定义清理参数

### 清理配置

在 `config.yaml` 中可以配置清理任务参数：

```yaml
# 清理任务配置
cleanup:
  # 清理任务执行间隔（分钟）
  interval_minutes: 10
  # 目录保留时间（分钟）
  retention_minutes: 10
```

- `interval_minutes`: 清理任务执行间隔，默认10分钟
- `retention_minutes`: 临时目录保留时间，默认10分钟

### 清理日志示例

```
2025/07/30 15:40:35 启动定时清理任务: 每10分钟清理一次超过10分钟的临时目录
2025/07/30 15:40:35 清理任务: 成功删除过期目录 coin_tmp_1753857838 (存在时间: 56m37s)
2025/07/30 15:40:35 清理任务完成: 共清理 3 个过期目录
```

## 错误处理

所有接口都包含统一的错误响应格式：

```json
{
  "success": false,
  "message": "错误描述信息"
}
```

常见错误：
- 400：请求参数格式错误
- 500：服务器内部错误（编译失败、网络错误等）

## 命令行调用

### 更新代币元数据

使用 BFC 客户端直接调用智能合约更新代币元数据：

```bash
/data/bfc client call --package 0xb405c1c029e436eac6ece68269c820528b9a28ee5615c7632bf70b5a6d705e72 --module metadata --function update_metadata --type-args 0xcc12500adf0f716436645b34c18ef5488b856429975256471680f32b6439bc6e::FASTCOIN::FASTCOIN --args 0xb081a303c5c7edae8d4f2c54e945154a651296f915c4a3c08e11c48a7651dbe1 [123,34,110,97,109,101,34,58,34,85,112,100,97,116,101,100,32,70,97,115,116,67,111,105,110,34,44,34,115,121,109,98,111,108,34,58,34,70,65,83,84,34,44,34,100,101,99,105,109,97,108,115,34,58,56,44,34,100,101,115,99,114,105,112,116,105,111,110,34,58,34,65,32,102,97,115,116,32,99,111,105,110,34,44,34,119,101,98,115,105,116,101,34,58,34,104,116,116,112,115,58,47,47,101,120,97,109,112,108,101,46,99,111,109,34,125]
```

### JSON 编码说明

命令中的 JSON 字符串需要编码成 16 进制数组格式。例如：

**原始 JSON：**
```json
{"name":"Updated FastCoin","symbol":"FAST","decimals":8,"description":"A fast coin","website":"https://example.com"}
```

**编码后的 16 进制数组：**
```
[123,34,110,97,109,101,34,58,34,85,112,100,97,116,101,100,32,70,97,115,116,67,111,105,110,34,44,34,115,121,109,98,111,108,34,58,34,70,65,83,84,34,44,34,100,101,99,105,109,97,108,115,34,58,56,44,34,100,101,115,99,114,105,112,116,105,111,110,34,58,34,65,32,102,97,115,116,32,99,111,105,110,34,44,34,119,101,98,115,105,116,101,34,58,34,104,116,116,112,115,58,47,47,101,120,97,109,112,108,101,46,99,111,109,34,125]
```

**编码方法：**
- 将 JSON 字符串的每个字符转换为对应的 ASCII 码
- 将 ASCII 码组成数组格式
- 在命令行中使用该数组作为参数

## 技术栈

- **后端框架：** Go + Gorilla Mux
- **智能合约：** Move 语言
- **编译工具：** BFC (Benfen Compiler)
- **网络：** Benfen 区块链网络

## 开发说明

项目结构：
```
├── config.go          # 配置文件处理
├── config.yaml        # 服务配置
├── handlers.go        # API 处理函数
├── main.go           # 服务入口
└── templates/        # 代币模板目录
```

主要功能模块：
- `addToken`: 代币模板生成和编译
- `publishToken`: 代币发布到区块链
- `compileMoveProject`: Move 项目编译
- `processTemplate`: 模板文件处理