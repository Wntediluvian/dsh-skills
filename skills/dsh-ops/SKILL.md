---
name: dsh-ops
description: >
  dsh（DeepSeek Harness）运维与故障排查技能。处理 dsh 无法启动、端口异常、
  插件加载失败、npm 发布异常、备份异常、进程管理、隐私保护等问题时加载本
  技能。触发词：排查问题、启动失败、3080 无法启动、dsh 报错、插件加载失败、
  npm 发布失败、备份异常、重启 dsh、dsh-ops。
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
| git push `schannel: SSL/TLS connection failed` | VPN/代理与 schannel 冲突 | 配代理：`git config --global http.proxy http://127.0.0.1:7897` |

## 手动命令

本技能无独立斜杠命令；直接说"排查问题"、"启动失败"、"dsh 报错"即可触发。

## 改动 dsh 插件后的必做清单

- [ ] 全局搜索旧包名（grep 插件目录，含 .yml 和 .js）
- [ ] `node --check lib/index.js` + `node --check lib/client.js`
- [ ] 三处一致性：git 仓库 / npm 包 / 本地安装目录哈希一致
- [ ] 告知用户重启，重启后验证：`/dsh-backup/status`、`/dsh-backup/config`、触发增量备份看 manifest
- [ ] **发布前隐私扫描**：确认无本机路径 / 用户名 / token / API key（见铁律 5）
