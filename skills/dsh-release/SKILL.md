---
name: dsh-release
description: >
  dsh（DeepSeek Harness）插件发布流程技能。涵盖版本检查、代码检查、内容
  检查、隐私检查、安全检查、本地验证、发布顺序（git + npm 整体更新）等
  完整流程。触发词：发布插件、发布流程、怎么发布、npm publish、版本号、
  更新插件、dsh-release。
---

# dsh 插件发布流程（dsh-release）

发布 dsh 插件的**标准流程**。完整手册：本目录下 `发布流程手册.md`
（脱敏公开版，随仓库分发）。本机完整版（含本机细节）仅保存在本地。

**发布原则：git 和 npm 必须整体同步更新，不可只发一个。**

## 发布前置：明确改动类型（决定版本号）

| 改动类型 | 版本号规则（SemVer） | 示例 |
|---|---|---|
| Bug 修复（向后兼容） | 修订号 +1 | 0.2.0 → 0.2.1 |
| 新功能（向后兼容） | 次版本 +1 | 0.1.1 → 0.2.0 |
| 破坏性变更 | 主版本 +1 | 1.0.0 → 2.0.0 |

## 一、代码检查

- [ ] `node --check lib/index.js` + `node --check lib/client.js`（exit=0）
- [ ] 改名场景查三层：`package.json name` = `cordis.patch.yml name` = `client.js load id` = 完整包名
- [ ] 三处一致性：git 仓库 / npm 包 / 本地安装目录哈希一致

## 二、版本检查

- [ ] `npm view <包> version` 确认线上版本 < 本地版本（npm 拒绝重复发布）
- [ ] CHANGELOG.md 已加对应条目（Added/Fixed/Changed 分类）

## 三、内容检查

- [ ] README：版本号、安装命令（`npm install @<作者>/<包>`）、功能说明最新
- [ ] 无"（待办）（当前环境）"等开发话术
- [ ] `npm pack --dry-run` 检查文件清单（只应有 lib/、README、CHANGELOG、package.json、patch）

## 四、隐私检查（重点）

- [ ] 扫描要发布的每个文件：盘符路径、用户目录、用户名、`ghp_`/`npm_` token、数据目录
- [ ] 含本机信息的文档只存本地，仓库放脱敏版

## 五、安全检查

- [ ] token 不写入任何公开文件、不进 git remote URL
- [ ] 插件无硬编码密钥（API key 走环境变量/配置文件）
- [ ] API key 文件在 NEVER 备份/还原集合中
- [ ] 插件 API 仅 loopback 响应

## 六、本地验证

- [ ] `/dsh-backup/status`、`/dsh-backup/config` 返回 200
- [ ] 触发增量备份，manifest 的 `plugins`/`skills` 正确
- [ ] 涉及加载改动：告知用户重启 dsh（不自行杀/启进程），重启后复测

## 七、发布顺序（严格按此顺序）

1. **git 提交并推送**：`git add -A && git commit -m "..." && git push`
2. **npm 发布**：`npm publish --access public`（作用域包必须）
3. **线上验证**：
   - 重复 publish 报 403 `cannot publish over previously published versions` = 已成功
   - 或 jsDelivr `https://cdn.jsdelivr.net/npm/<包>@<版本>/package.json` HTTP 200
   - `npm view <包> version` 确认（注意 CDN 延迟，别慌）

## 八、网络坑速查

| 症状 | 处理 |
|---|---|
| `schannel: SSL/TLS connection failed` | 配代理 `git config --global http.proxy http://127.0.0.1:7897` |
| github.com 直连失败 | 走代理或切换 VPN 模式 |

## 口诀

> 版本要对、语法要过、三层要齐、文档要新、隐私要扫、安全要查、先推 git、再发 npm、延迟别慌、jsDelivr 验证。
