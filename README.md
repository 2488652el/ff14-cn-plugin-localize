<div align="center">

# 🌙 ff14-cn-plugin-localize

**FFXIV 卫月（Dalamud）插件简体中文汉化 · AI Agent Skill**

从 6 个真实汉化项目中提炼的可复用工作流，交给 AI 一句话开工

[![Skill](https://img.shields.io/badge/type-Agent%20Skill-blueviolet?style=flat-square)](SKILL.md)
[![Lang](https://img.shields.io/badge/lang-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-red?style=flat-square)](SKILL.md)
[![Plugins](https://img.shields.io/badge/%E5%AE%9E%E8%B7%B5%E9%A1%B9%E7%9B%AE-6%20%E4%B8%AA%E5%B7%B2%E5%8F%91%E5%B8%83%E6%B1%89%E5%8C%96-green?style=flat-square)](https://github.com/2488652el/dalamud-plugins)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

</div>

---

## ✨ 这是什么

一个给 AI 编程助手（Kimi Code / Claude Code 等支持 Skill 的 Agent）使用的**技能文件**。装上之后，只需说一句"帮我汉化这个卫月插件"，AI 就会按这套被实战验证过的流程执行：判断策略 → 翻译 → 排坑 → 迁移配置 → 打包发布。

内容提炼自以下已发布的汉化项目，全部经国服客户端实测：

| 插件 | 汉化策略 | 规模 |
|---|---|---|
| [Allagan Tools](https://github.com/2488652el/dalamud-plugins) | 就地硬改，独立 commit | 543 文件 / 4000+ 处 |
| visland-cn | 就地硬改 + 共存改名 | 34 文件 |
| Browsingway-cn | 就地硬改 | 60+ 处 |
| ItemVendorLocation-cn | 扩展 CheapLoc 双语字典 | 2 本字典 |
| Submarine Tracker-cn | 复用上游 resx，只补 zh | 2 文件 17 行 |
| CharacterPanelRefined | 复用上游 resx（待做） | 70 key |

## 🚀 安装

把整个目录复制到你的 Agent skills 目录即可：

```bash
# Kimi Code / 通用 user 级
git clone https://github.com/2488652el/ff14-cn-plugin-localize.git
cp -r ff14-cn-plugin-localize ~/.agents/skills/ff14-cn-plugin-localize

# 或项目级
cp -r ff14-cn-plugin-localize /path/to/project/.agents/skills/
```

> Windows 用户目录示例：`C:\Users\<你>\.agents\skills\ff14-cn-plugin-localize\`

## 🎯 使用

对 AI 说出类似的话即可触发：

- 「汉化这个 Dalamud 插件」
- 「把这个卫月插件做成简体中文版」
- 「上游更新了，帮我同步汉化」

## 🧭 核心方法论速览

<details open><summary><b>第 0 步：三种策略，按优先级判断</b></summary>

1. **上游有 i18n 体系（resx / Crowdin）** → 零代码，只补 `*.zh.resx`
2. **上游有简陋本地化骨架（CheapLoc 等）** → 扩展成中英双语字典 + `Loc.Localize(key, fallback)`
3. **纯硬编码字符串（最常见）** → 字面量 1:1 替换，收敛为独立 commit 便于 rebase

</details>

<details><summary><b>三条「不用管」</b></summary>

- 🔤 **字体** — 国服卫月自带 CJK 支持，插件零字体代码
- 🗃️ **游戏数据文本** — Lumina 读国服客户端天然中文，绝不自建翻译表
- 📐 **布局** — 中文比英文短，一般无需调尺寸

</details>

<details><summary><b>不能翻的红线</b></summary>

`InternalName` / 命令名 / IPC 频道 / 配置 Key / ImGui `##`·`###` ID 锚点 / `{0}` 占位符 / 嵌入数据 JSON

</details>

<details><summary><b>💣 最大的坑：对游戏文本做字符串匹配的代码</b></summary>

tooltip 前缀检测、按英文名解析游戏菜单、物品名等值比较——换中文客户端后**全部失效**，语序差异会把 `==` 逼成 `StartsWith`。汉化后真正要逐行排查的是它们，不是 UI。

</details>

完整内容见 [SKILL.md](SKILL.md)：含国服语言枚举 `(ClientLanguage)4`、`.WithLanguage()` 锁英文读法、存量配置迁移范式、发布到第三方仓库的版本一致性清单、验收清单与维护提示。

## 📂 仓库结构

```
ff14-cn-plugin-localize/
├── SKILL.md    # 技能本体（Agent 读取）
└── README.md   # 本文件
```

## 🙏 致谢

- 汉化实践：[@2488652el](https://github.com/2488652el) 的[卫月插件仓库](https://github.com/2488652el/dalamud-plugins)
- 各插件上游作者：Critical-Impact / Styr1x / electr0sheep / Infiziert90 / awgil / Kouzukii

## 📄 License

[MIT](LICENSE)
