# 360 全家桶残留安全清理 Skill

> WorkBuddy / CodeBuddy 技能：安全清理 Windows 上 360 全家桶的安装目录、用户数据、注册表键、服务与失效驱动残骸。

一个"先看清、再确认、可回滚"的 360 清理技能。不是粗暴的 `rm -rf`，而是把 2026-11-22 一次真实清理中踩过的坑（沙箱限制、回收站兼容性、卸载程序空等服务卡死）固化成的可复用流程。

## 为什么需要它

360 卸载后往往留下大量残骸：程序目录、藏在 `secoresdk` 容器里的浏览器数据、改了后缀失效的驱动、注册表键和服务项。更麻烦的是——**残留状态下直接拉起官方 `uninst.exe` 会卡死**：它会在一个已经不存在的 `360Box64` 服务上空等（`Waiting for service ... to stop` 无限刷屏）。

这个技能的核心判断就是区分两种情况：

| 场景 | 打法 |
|---|---|
| **活跃安装**（注册表有卸载登记、服务在跑） | 先走官方卸载，再清残留 |
| **纯残骸**（登记已空、只剩文件和注册表壳） | 直接文件级清理，**不拉 uninst.exe** |

## 功能特性

- **只读扫描**：一条命令产出 JSON 清单，含体积、是否需提权、置信度（确认/存疑/误报）。
- **三态判定**：`active` / `residue` / `clean`，自动决定该走卸载还是直接清理。
- **指纹库驱动**：覆盖 11 个注册表键、4 个服务、6 类失效驱动后缀、`Wukong` 捆绑游戏识别、文件名误报排除。
- **可回滚**：优先移入回收站；系统保护位置降级为隔离目录；注册表先导出 `.reg` 再删。
- **沙箱适配**：明确记录当前环境被拦的调用与绕法，做不到的生成提权脚本交用户，不假装完成。

## 安装

作为 WorkBuddy 用户技能安装（跨项目可用）：

```bash
# 方式 A：git clone 到技能目录
git clone https://github.com/oracis/360-residue-cleanup.git \
  ~/.workbuddy/skills/360-residue-cleanup

# 方式 B：下载 zip 解压到 ~/.workbuddy/skills/360-residue-cleanup/
```

安装后，对话里说"清理360""卸载360""清掉360残留"即可自动触发。

## 使用

技能会引导完整流程，也可直接调用脚本：

```powershell
# 1. 只读扫描（不删任何东西），产出 JSON 清单
powershell -ExecutionPolicy Bypass -File scripts/scan_360.ps1 -OutFile .\scan-result.json

# 2. 清理（需管理员权限；可用扫描结果驱动，或用内置默认清单）
powershell -ExecutionPolicy Bypass -File scripts/cleanup_360_admin.ps1
```

⚠️ **清理脚本需右键"以管理员身份运行"**。`C:\Program Files`、`System32\drivers` 和注册表在当前沙箱下无权限直接操作，脚本会先备份、再逐步清理，并把删不掉的项列成手动清单。

## 安全设计

1. 先只读扫描出清单，再谈删除。
2. 删除必须经用户逐项确认（尤其 `Wukong` 和浏览器书签）。
3. 先备份（书签 + 注册表 `.reg`），后删除。
4. 可回滚优先：回收站 / 隔离区，绝不默认永久删除。
5. 分批次、逐批验证。
6. 环境做不到如实报告，生成提权脚本，不强绕安全策略。
7. 残骸态绝不拉 `uninst.exe`。

## 目录结构

```
360-residue-cleanup/
├── SKILL.md                        # 技能主文件（frontmatter + 流程 + 安全铁律）
├── scripts/
│   ├── scan_360.ps1                # 只读扫描 → JSON 清单（含三态判定）
│   └── cleanup_360_admin.ps1       # 需管理员的清理脚本
└── references/
    ├── 360-fingerprints.md         # 指纹库：目录/注册表/服务/驱动/误报排除
    └── sandbox-limits.md           # 沙箱限制与绕法（实测记录）
```

## 发布状态

- SkillHub：`360-residue-cleanup`（IT 运维与安全 / 故障排查）
- ClawHub：`360-residue-cleanup`

## License

MIT
