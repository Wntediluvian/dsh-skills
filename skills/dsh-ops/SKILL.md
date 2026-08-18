---
name: dsh-ops
description: >
  dsh（DeepSeek Harness）运维与故障排查技能。处理 dsh 无法启动、端口异常、
  插件加载失败、npm 发布异常、备份异常、进程管理、隐私保护等问题时加载本
  技能。触发词：ops、排查问题、启动失败、3080 无法启动、dsh 报错、插件加载
  失败、npm 发布失败、备份异常、重启 dsh、dsh-ops。只要用户提到 ops 即触发。
---

# dsh 运维（dsh-ops）

dsh 运维与故障排查的**快速入口**。通用版故障档案：本目录下 `故障处理经验库.md`
（脱敏公开版，随仓库分发）。本机完整经验库（含个人路径）仅保存在本地。

处理任何 dsh 相关故障时：先看铁律 → 对照定位表 → 执行检查清单 → 必要时读完整档案。

## 铁律（必须遵守）

1. **绝不自行杀/启 dsh 进程**：dsh 主进程由用户手动管理。需要重启时，改好文件后告知用户"请重启 dsh"，由用户操作，我再用 API 验证。
2. **插件改名是三层系统**：`package.json` 的 `name`、`cordis.patch.yml` 的 `name`、`client.js` 的 `__ModuleLoader__.load({ id })` **必须同步**。规则：**patch name / client id = 完整包名**（如 `@<作者>/<包>`），代码内注册名（`export const name`、slots id、API 路径）可保留短名。参照 modlens 模式。
3. **npm 发布后 404 是延迟不是失败**：发布命令 exit=0 即成功。用"重复发布报 403 已存在"或 jsDelivr（`https://cdn.jsdelivr.net/npm/<包名>@<版本>/package.json` HTTP 200）验证，不要用 `npm view` 立即判断。
4. **作用域包发布必须** `npm publish --access public`（否则 E402 private packages）。
5. **隐私保护（重要）**：本机路径、用户名、token、API key、数据目录结构**绝不写入**要发布到公开仓库的文件。含个人信息的经验库只存本地。发布前必须扫描：盘符路径、用户目录、用户名、`ghp_`/`npm_` token、数据目录等。
6. **改 node_modules 的脚本必须字节安全**：PS 5.1 的 `Get-Content`/`Set-Content` 按 ANSI 处理会破坏 UTF-8 中文。一律用 `[System.IO.File]::ReadAllText/WriteAllText`（UTF8 无 BOM），.ps1 脚本本身存 **UTF-8 带 BOM**。补丁脚本参考 `运维\插件补丁\reapply-patches.ps1`。

## 故障快速定位表

| 症状 | 根因 | 处理 |
|---|---|---|
| 3080 启动报 `Cannot find package 'xxx'` | cordis.patch.yml 的 name 与包名不符 | 改为完整包名 |
| 浏览器报 `loaded without registering "@xxx"` | client.js 的 load id 与包名不符 | 改为完整包名 |
| `npm publish` E402 | 作用域包默认私有 | 加 `--access public` |
| `npm publish` E403 2FA | token 无 bypass 2FA | 用 .npmrc 里已配置的 token |
| `npm publish` EOTP | 需要一次性验证码 | `--otp=<动态码>`（问用户要，不是 PIN） |
| 发布后 `npm view` 404 | CDN 传播延迟 | 重复发布报 403 = 已成功，或用 jsDelivr 验证 |
| 备份 `plugins.files=0` | 增量备份文件未变化（正常） | 骨架目录仍建立；全量才复制全部 |
| git push `schannel: SSL/TLS connection failed` | VPN/代理与 schannel 冲突 | 配代理：`git config --global http.proxy <代理端口>` |
| 插件配置页卡片不显示（改 key 也没用）| 没注册 **settings namespace**（页只渲染有 namespace 的插件）| 服务端 `installSettingsSection(ctx, settingsNamespace('<名>'), z.object({}), ...)` |
| PowerShell 窗口反复弹出 | dsh 进程**无控制台**（detached 拉起）→ 子进程各自弹窗 | 用 `spawn(..., { detached:true, windowsHide:true })` 重启；**勿用 powershell 包装**（会重启失败）|
| 重启按钮触发后拉不起来 | 重启命令失败（如 powershell 包装）| 回退 `detached + windowsHide`；查 `%TEMP%/dsh-restart-*.log` |
| 插件市场更新后功能丢失/报错 | 一键更新 = `pnpm add @latest` **重装整个包目录**，node_modules 手动补丁全被覆盖 | 更新后跑 `运维\插件补丁\reapply-patches.ps1` 重打补丁（幂等）；自持插件用 `file:`/`github:` 安装可永久免疫 |
| 3080 启动失败 `Failed to load plugins` / `bundle script ... failed to load` | `pnpm install` 清空了不在 dependencies 里的 `@deepseek-ai/*` 官方 bundle | package.json dependencies 补 `@deepseek-ai/dsh-base` + `dsh-web-app`（^0.1.0-rc.7）→ `pnpm install --no-frozen-lockfile` 重装 → 跑 reapply-patches.ps1 |
| 启动报 `keyed slot "settings.plugin.item" requires options.key`（具体插件名） | 该插件 client.js 用 `id` 注册 keyed slot（应为 `key`）| 改 `id:` → `key:`；已在 modlens/aqua/dsh-restart/dsh-backup 遇到，`reapply-patches.ps1` 已覆盖 modlens+aqua |
| 模型选择器锁死，只能选 `(modlens vision)` 变体 | **会话里有图片附件** → dsh 规则禁止切纯文本模型（不是 bug）| 开新会话即解锁；视觉插件二选一（modlens/vision-router 会抢路由）|
| `dsh plugin update/add @latest` 后版本没变 | **pnpm 24h 发布冷静期**（minimumReleaseAge）拦了新版本 | 加 `--config.minimumReleaseAge=0` 绕过 |
| 响应整体变慢 | ①会话过长（>1000 步/10 万 token 上下文）②vision-router stealth 接管 deepseek 路由 ③modlens 变体包装层 | 开新会话；`vision-router` 设 `stealth: false` 或卸载；默认模型用 `deepseek-official` 而非 `deepseek-modlens` |
| settings 卡片报 `props.useXxx is not a function` | hooks key **带了 `use` 前缀**（dsh 自动从 `xxx` 铸 `useXxx`）| hooks key 用 `dshDock` 而非 `useDshDock` |
| npm 发布后 pnpm 装不上（404）| dist-tags 为空 / CDN 延迟 | `npm dist-tag add <pkg>@<ver> latest`；或重复 publish 报 403 验证 |

## 手动命令

本技能无独立斜杠命令；只要提到 **"ops"** 即触发，或直接说"排查问题"、"启动失败"、"dsh 报错"等触发词。

## 开发契约速查（插件开发易错点）

### 服务端（lib/index.js）
- 取 webServer：`ctx.inject(['webServer'], (scope) => { scope.webServer ... })`（直接 `ctx.webServer` 抛错）
- 函数定义必须在引擎闭包内；暴露用 `engine.xxx`
- 变量名避免与函数名相同（TDZ 遮蔽）

### 客户端（lib/client.js）
- **必须导出** `exports.apply(ctx)`；`exports.inject = ['slots']` 用**顶层**声明（scoped 不生效）
- 取 React：`require('react')`（不是 `window.React`）
- `__ModuleLoader__.load({ id: '<完整包名>' })` —— id 必须 = npm 包名
- **settings 卡片 slots.register 的 hooks key 不能带 `use` 前缀**：dsh 框架自动从 `dshDock` 铸出 `props.useDshDock`。写 `hooks: { useDshDock: store }` 会拿到原始 store（报 `props.useDshDock is not a function`）。参照原版 dsh-restart：`hooks: { dshRestart: store }` → `props.useDshRestart`

### 通用
- 时间戳目录名无冒号（Windows 禁止）；`new Date('无冒号串')` 是 Invalid Date → 加 `isNaN` 容错
- bat 脚本用英文注释 + GBK 编码（中文 rem 会让 cmd 崩溃）
- 写中文文件用 write 工具（PowerShell Set-Content 会 GBK 乱码）

## 技能自动更新（重要机制）

**触发更新时机**（用户说"更新 skills"或以下任一情况，应立即更新本技能 + 经验库 + 仓库）：
1. 处理完任何重大事故/新报错后
2. 学到新的开发契约/环境坑后
3. 发布流程有变化后
4. 用户明确说"更新 skills"

**更新流程**：本机经验库（本地完整版）→ 本机技能 → 仓库脱敏版（`git/dsh-skills`）→ git 提交推送。

## 改动 dsh 插件后的必做清单

- [ ] 全局搜索旧包名（grep 插件目录，含 .yml 和 .js）
- [ ] `node --check lib/index.js` + `node --check lib/client.js`
- [ ] 三处一致性：git 仓库 / npm 包 / 本地安装目录哈希一致
- [ ] 告知用户重启，重启后验证：`/dsh-backup/status`、`/dsh-backup/config`、触发增量备份看 manifest
- [ ] **发布前隐私扫描**：确认无本机路径 / 用户名 / token / API key（见铁律 5）
