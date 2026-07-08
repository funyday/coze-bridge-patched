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

## Agent映射表（示例）
| Coze云端Agent ID | 角色说明 | OpenClaw Agent ID |
|---|---|---|
| `<coze-agent-id-1>` | Agent角色A | `<openclaw-agent-id-1>` |
| `<coze-agent-id-2>` | Agent角色B | `<openclaw-agent-id-2>` |
| `<coze-agent-id-3>` | Agent角色C | `<openclaw-agent-id-3>` |
| `<coze-agent-id-4>` | Agent角色D | `<openclaw-agent-id-4>` |

> 实际使用时，将 `<coze-agent-id-N>` 替换为 Coze 云端 Agent 的真实 ID。

## 文件说明
| 文件 | 说明 |
|---|---|
| lib/index.js | v3+v4 patched bridge lib (506,122 bytes) |
| lib/index.js.bak | 原始未修改备份 (505,789 bytes) |
| lib/package.json | 原始 package.json |
| local-agent-mapping.json | Agent映射配置模板 |

## 部署步骤
1. 停止现有daemon: `coze-bridge stop`
2. 备份原始lib: `cp lib/index.js lib/index.js.bak`
3. 替换lib: `cp patched/lib/index.js lib/index.js`
4. 部署映射文件: `cp local-agent-mapping.json ~/.coze/local-agent-mapping.json`
5. 编辑映射文件，填入真实的 Coze Agent ID 和 OpenClaw Agent ID
6. 清除所有agent的持久化会话（config.json中sessions字段设为`{}`）
7. 重启daemon: `coze-bridge stop`（supervisor自动重启）

## 注意事项
- daemon由launchd supervisor管理，`kill`无效，必须用`coze-bridge stop`
- 映射文件路径: `~/.coze/local-agent-mapping.json`，daemon不会修改此文件
- `localAgentId`必须在pairing之后添加（pair时会重写config.json）
- 编辑config.json必须用vim/VS Code/nano，禁止用Mac自带文本编辑器（弯引号问题）
- 仓库中不包含任何敏感配置（config.json、pat-token、bridge.token 已通过 .gitignore 排除）
