# dsh-skills

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 技能合集。

## 技能列表

| 技能 | 说明 | 触发词 |
|---|---|---|
| [dsh-ops](./skills/dsh-ops) | dsh 运维与故障排查：启动失败、插件加载、npm 发布异常、备份异常 | 排查问题、启动失败、3080 报错、插件加载失败、备份异常 |
| [dsh-release](./skills/dsh-release) | 插件发布流程：版本检查、代码/内容/隐私/安全检查、发布顺序 | 发布插件、发布流程、npm publish、怎么发布 |

## 安装

将 `skills/` 下的技能目录复制到 dsh 的技能根目录：

```bash
git clone https://github.com/Wntediluvian/dsh-skills.git
# 复制全部技能
cp -r dsh-skills/skills/* $DSH_HOME/skills/
# 或按需复制单个技能
cp -r dsh-skills/skills/dsh-ops $DSH_HOME/skills/
```

Windows PowerShell：

```powershell
git clone https://github.com/Wntediluvian/dsh-skills.git
Copy-Item dsh-skills/skills/* -Destination $env:DSH_HOME/skills/ -Recurse -Force
```

重启 dsh 后生效。

## 更新技能

```bash
cd dsh-skills && git pull
Copy-Item skills/* -Destination $env:DSH_HOME/skills/ -Recurse -Force
```

## 内容说明

- `skills/dsh-ops/`：运维与故障排查经验（SKILL.md 为快速入口；`故障处理经验库.md` 为通用脱敏版档案，不含任何本机路径）
- `skills/dsh-release/`：发布流程手册（SKILL.md 为快速入口；`发布流程手册.md` 为完整版）

## 隐私声明

本仓库所有文件均为**脱敏公开版**，不含本机路径、用户名、token 等隐私信息。
含个人细节的经验库仅保存在使用者本地。

## License

MIT
