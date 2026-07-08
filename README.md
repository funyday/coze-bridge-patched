# Coze-Bridge 多Agent路由改造

## 版本信息
- 基于 coze-bridge v0.2.2
- 改造日期: 2026-07-07
- 改造目标: 实现Coze云端多个本地Agent分别路由到不同的OpenClaw本地Agent

## 改造内容

### v3 patch: 独立映射文件 + workspace路径计算
在 `buildOpenclawDownstream` 方法中添加本地Agent映射逻辑：
- 读取 `~/.coze/local-agent-mapping.json` 映射文件（`{"<cozeAgentId>": "<openclawAgentId>"}`）
- 根据映射计算正确的workspace路径：`workspace-{openclawAgentId}`
- 用正确的CWD和agentId构造 `Br` 对象

### v4 patch: ensureDownstream CWD修复
将 `ensureDownstream` 方法中3处 `cwd:e.workspace` 替换为 `cwd:h.opts.cwd`：
- `h` 是 `Br` 对象（由 `buildOpenclawDownstream` 通过v3 patch创建）
- `h.opts.cwd` 是v3 patch设置的正确CWD
- 确保 `session/new` 和 `session/load` 都使用正确的workspace路径

## Agent映射表
| Coze云端Agent ID | 显示名称 | OpenClaw Agent ID |
|---|---|---|
| 7659331842738290953 | 果子IT工程师 | it-engineer |
| 7647145924141089059 | 果子-网络安全项目经理 | cybersec-manager |
| 7659332122796294450 | 果子网络安全审批人 | cs-approver |
| 7659332465185669403 | 果子网络安全顾问 | cs-assistant |

## 文件说明
| 文件 | 说明 |
|---|---|
| lib/index.js | v3+v4 patched bridge lib (506,125 bytes) |
| lib/index.js.bak | 原始未修改备份 |
| local-agent-mapping.json | Agent映射配置文件 |
| test-acp.js | ACP协议测试脚本 |

## 部署步骤
1. 停止现有daemon: `coze-bridge stop`
2. 备份原始lib: `cp lib/index.js lib/index.js.bak`
3. 替换lib: `cp patched/lib/index.js lib/index.js`
4. 部署映射文件: `cp local-agent-mapping.json ~/.coze/local-agent-mapping.json`
5. 清除所有agent的持久化会话（config.json中sessions字段设为`{}`）
6. 重启daemon: `coze-bridge stop`（supervisor自动重启）

## 注意事项
- daemon由launchd supervisor管理，`kill`无效，必须用`coze-bridge stop`
- 映射文件路径: `~/.coze/local-agent-mapping.json`，daemon不会修改此文件
- `localAgentId`必须在pairing之后添加（pair时会重写config.json）
- 编辑config.json必须用vim/VS Code/nano，禁止用Mac自带文本编辑器（弯引号问题）
