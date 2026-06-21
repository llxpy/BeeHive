# BeeHive（蜂巢）——全自动桌面接管系统

## 🐝 概述

BeeHive 是一个基于蜂群隐喻的桌面级全自动化软件，能够接管用户的所有操作。它采用 **蜂后-工蜂-守卫蜂** 三层架构，通过并行工蜂池提高效率，守卫蜂实时监控确保安全，蜂巢层提供系统级接管能力。

**核心特性**：
- 🏗️ **蜂群架构**：蜂后（LLM推理）+ 工蜂（执行）+ 守卫蜂（监控）+ 蜂巢（系统接管）
- 🔐 **分级授权**：L0-L4 五级权限，启动前必须用户明确授权
- 🚨 **实时监管**：守卫蜂独立监控，异常工蜂直接 kill
- 📝 **中文日志**：所有操作结构化记录，每日轮转
- ⚡ **并行执行**：多工蜂同时工作，支持蜂舞协议传递中间结果
- 🛡️ **执行即销毁**：工蜂在隔离目录执行，完成后目录自动清理
- 🖥️ **系统接管**：窗口管理、输入模拟、进程控制、注册表操作等

## 📁 项目结构

```
BeeHive/
├── beehive.py              # 入口：GUI 权限授权 + 系统托盘监控
├── queen.py                # 蜂后：LLM推理 + 任务调度
├── worker.py               # 工蜂：克隆执行体（单文件，零依赖）
├── guard.py                # 守卫蜂：实时监控 + 异常 kill
├── hive.py                 # 蜂巢：Windows 系统接管层
├── permission.py           # 权限系统：L0-L4 分级 + 签名
├── honeycomb.py            # 日志系统：中文结构化日志
├── local_config.json       # 本地配置文件（API 密钥等）
├── config.json             # 配置模板（参考）
├── memory/
│   ├── hive_knowledge.md   # 固化知识库
│   └── hints.md            # 记忆线索
├── sessions/               # 会话持久化目录
├── clone_pool/             # 工蜂临时工作目录池
└── logs/                   # 中文日志目录
```

## 🚀 快速开始

### 1. 配置 API 密钥

编辑 `local_config.json`，填入你的 LLM API 信息：

```json
{
  "api": {
    "base_url": "https://api.deepseek.com/v1",
    "model_name": "deepseek-chat",
    "api_key": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  },
  "agent": { ... },
  "security": { ... },
  "hive": { ... }
}
```

支持任何 OpenAI 兼容 API（DeepSeek、OpenAI、Azure 等）。

### 2. 运行方式

#### GUI 模式（推荐）
```bash
python beehive.py
```
1. 弹出权限授权窗口
2. 选择权限等级（L0-L4）
3. 确认授权 → 启动蜂后
4. 进入系统托盘监控界面

#### 命令行模式
```bash
# 跳过 GUI，直接启动（默认 L2 权限）
python beehive.py --auto

# 指定权限等级（L0-L4）
python beehive.py --auto --level L3

# 单任务模式
python beehive.py --task "整理桌面文件"
```

## 🔐 权限等级

| 等级 | 名称 | 权限范围 | 适用场景 |
|------|------|----------|----------|
| **L0** | 只读 | 文件读取、系统信息查询 | 安全审计、信息查询 |
| **L1** | 沙箱 | `~/.beehive/sandbox` 内任意操作 | 测试、实验 |
| **L2** | 用户空间 | 桌面/文档/下载等用户目录读写 | 日常文件管理、文档处理 |
| **L3** | 系统级 | 全磁盘访问 + 进程管理 + 注册表（HKCU） | 系统优化、程序管理 |
| **L4** | 完全接管 | 关机/重启/网络配置/驱动安装/服务管理/注册表（HKLM） | 完全自动化 |

**注意**：权限越高，BeeHive 能执行的操作越多，但风险也越大。建议从 L2 开始。

## 🐝 蜂群角色说明

### 蜂后 (Queen)
- **职责**：LLM 推理、任务分解、工蜂调度
- **特点**：绝不触碰 Shell，只通过 `spawn_worker` 派发任务
- **深度限制**：depth=0（蜂后），depth=1（工蜂），depth≥2 硬拦截
- **记忆系统**：`hive_knowledge.md`（固化知识）+ `hints.md`（记忆线索）

### 工蜂 (Worker)
- **职责**：执行具体命令
- **特点**：单文件、零依赖、执行即销毁
- **危险命令过滤**：拦截 `format`、`del /f/s`、`rm -rf`、`shutdown` 等
- **蜂舞协议**：同批次工蜂可通过 `.bee_dance.json` 传递中间结果

### 守卫蜂 (Guard)
- **职责**：监控所有工蜂，异常时 kill
- **监控指标**：
  - CPU 使用率 >80% 持续 10 秒
  - 内存使用 >500MB 且持续增长
  - 文件访问越权（超出白名单）
  - 执行超时（超过预设 2 倍）
  - 网络连接可疑
- **Kill 流程**：SIGTERM → 3 秒 → SIGKILL → 清理目录 → 通知蜂后

### 蜂巢 (Hive)
- **职责**：Windows 系统接管层
- **能力**：
  - 窗口管理（移动、最小化、最大化）
  - 输入模拟（键盘、鼠标）
  - 进程管理（启动、终止、监控）
  - 文件系统（全盘读写 + 监控）
  - 注册表操作（HKCU/HKLM）
  - 服务管理（启动、停止、配置）
  - 网络配置（DNS、代理、防火墙）
  - 计划任务（创建、删除、执行）

## 📝 日志系统（蜜脾）

### 日志结构
```
logs/
└── 2026-06-03/
    ├── index.json          # 全天操作索引
    ├── queen.log           # 蜂后决策日志
    ├── guard.log           # 守卫蜂监控日志
    ├── worker_b3a2.log     # 工蜂操作日志
    └── ...
```

### 日志格式
```
[2026-06-03 14:32:01] [工蜂-7a3f] [文件操作] 复制 D:\docs\报告.docx → D:\backup\报告.docx | 耗时 0.3s | 结果 成功
[2026-06-03 14:32:05] [守卫蜂-01] [异常检测] 工蜂-7a3f 试图访问 C:\Windows\System32 → 已拦截，风险等级: 高
[2026-06-03 14:32:06] [守卫蜂-01] [终止] 工蜂-7a3f(PID 12345) 已终止，原因: 越权访问系统目录
```

### 日志轮转
- 每天创建新目录（`logs/YYYY-MM-DD/`）
- 保留最近 30 天日志
- 每日索引文件汇总当天所有操作和风险事件

## ⚠️ 重要注意事项

### 1. 安全警告
- **BeeHive 拥有系统级访问权限**，错误使用可能导致数据丢失或系统损坏
- 守卫蜂 kill 机制是最后防线，不能替代用户监督
- 危险命令过滤仅为基础防护，复杂恶意命令可能绕过
- 建议在虚拟机或测试环境中首次使用

### 2. 权限管理
- **启动前必须授权**：GUI 模式强制用户选择权限等级
- **权限不可逆升级**：低权限无法执行高权限操作
- **签名验证**：权限文件使用 HMAC 签名，防止篡改
- **白名单机制**：根据权限等级自动生成路径白名单

### 3. 资源使用
- **工蜂池上限**：默认 8 个并行工蜂，可在配置中调整
- **内存监控**：守卫蜂监控工蜂内存使用，超 500MB 自动 kill
- **CPU 监控**：持续高 CPU 使用（>80%）触发 kill
- **超时保护**：工蜂默认超时 300 秒，超时 2 倍强制终止

### 4. 系统兼容性
- **操作系统**：Windows 10/11（主要），Linux/macOS（核心功能可用）
- **Python 版本**：Python 3.8+
- **依赖**：零第三方依赖，仅 Python 标准库
- **管理员权限**：L3/L4 权限需要管理员权限运行

### 5. 故障处理
- **蜂后崩溃**：蜂巢守护进程自动重启（最多 3 次）
- **守卫蜂失联**：蜂后检测到心跳超时后重新启动守卫蜂
- **工蜂卡死**：守卫蜂监控超时自动 kill
- **日志系统故障**：日志写入失败不影响主流程

### 6. 隐私与数据
- **本地运行**：所有代码在本地执行，不发送用户数据到远程服务器（除 LLM API 调用）
- **日志包含敏感信息**：日志中可能包含文件路径、命令内容等
- **记忆持久化**：会话历史保存在 `sessions/` 目录
- **临时文件清理**：工蜂工作目录执行后自动清理

## 🔧 配置说明

### `local_config.json` 详解

```json
{
  "api": {
    "base_url": "https://api.deepseek.com/v1",  // API 地址
    "model_name": "deepseek-chat",              // 模型名称
    "api_key": ""                               // API 密钥（必填）
  },
  "agent": {
    "max_depth": 3,                             // 最大深度（≥3 硬拦截）
    "max_workers": 8,                           // 最大工蜂数
    "worker_timeout": 300,                      // 工蜂超时（秒）
    "compact_threshold": 0.85,                  // 记忆压缩阈值（0.85=85%）
    "token_cap": 100000                         // Token 容量上限
  },
  "security": {
    "permission_level": "L2",                   // 默认权限等级
    "guard_check_interval": 5,                  // 守卫蜂检查间隔（秒）
    "cpu_threshold": 80,                        // CPU 阈值（%）
    "memory_threshold_mb": 500,                 // 内存阈值（MB）
    "kill_grace_seconds": 3                     // Kill 等待时间（秒）
  },
  "hive": {
    "auto_start": false,                        // 开机自启
    "daemon_mode": true,                        // 守护模式
    "max_queen_restarts": 3                     // 蜂后最大重启次数
  }
}
```

### 环境变量覆盖
配置优先级：**环境变量 > local_config.json > config.json > 硬编码默认值**

| 环境变量 | 对应配置项 | 示例 |
|----------|------------|------|
| `BEEHIVE_API_KEY` | `api.api_key` | `set BEEHIVE_API_KEY=sk-xxx` |
| `BEEHIVE_MODEL_NAME` | `api.model_name` | `set BEEHIVE_MODEL_NAME=gpt-4` |
| `BEEHIVE_MAX_WORKERS` | `agent.max_workers` | `set BEEHIVE_MAX_WORKERS=12` |
| `BEEHIVE_PERMISSION_LEVEL` | `security.permission_level` | `set BEEHIVE_PERMISSION_LEVEL=L3` |

## 🐛 常见问题

### Q1: 启动时报 "API Key 未配置"
A: 在 `local_config.json` 中填写 `api.api_key`

### Q2: 工蜂执行命令被拦截
A: 检查权限等级是否足够，危险命令（format/del/rm/shutdown）会被过滤

### Q3: 守卫蜂频繁 kill 工蜂
A: 调整 `security.cpu_threshold` 或 `security.memory_threshold_mb`

### Q4: 蜂后响应慢
A: 检查网络连接，或降低 `agent.max_workers` 减少并行数

### Q5: 日志文件过大
A: 日志自动保留 30 天，可手动删除 `logs/` 目录下旧文件夹

### Q6: 如何完全卸载
A: 删除 BeeHive 文件夹，检查以下位置：
- 注册表：`HKCU\Software\Microsoft\Windows\CurrentVersion\Run\BeeHive`
- 计划任务：`schtasks /delete /tn "BeeHive" /f`
- 环境变量：移除 `BEEHIVE_` 开头的变量

## 📄 许可证与免责声明

### 许可证
本项目基于 MIT 许可证开源。

### 免责声明
**重要**：BeeHive 是一个实验性项目，作者不对使用本软件造成的任何损失负责。包括但不限于：
- 数据丢失或损坏
- 系统不稳定或崩溃
- 安全漏洞导致的损失
- 违反法律法规的后果

**使用前请务必**：
1. 在测试环境中验证功能
2. 备份重要数据
3. 理解各权限等级的风险
4. 仅在必要时使用高权限等级

## 🔗 相关项目

- **AntNest**：单文件 LLM Agent，BeeHive 的灵感来源
- **EVA**：开源 LLM Agent 框架
- **Auto-GPT**：早期自动化 AI 代理

## 📞 支持与反馈

如有问题或建议：
1. 查看 `logs/` 目录下的详细日志
2. 检查是否满足系统要求
3. 在项目仓库提交 Issue

---

**最后提醒**：能力越大，责任越大。请负责任地使用 BeeHive。

*"像蜂群一样协作，像守卫一样警惕，像蜂后一样智慧。"*



/**
注意因为蜂巢在创建文件总是使用GBK来进行创建，但是这样的写法会导致编码问题。
特别是中文，所以在使用的时候，请让它自己，将使用python进行读写操作时的编码进行UTF-8的约束。

**/
