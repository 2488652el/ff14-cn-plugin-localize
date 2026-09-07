---
name: ff14-cn-plugin-localize
description: 汉化 FFXIV 卫月（Dalamud）插件为简体中文并发布到自建第三方仓库的标准流程。涵盖三种策略（复用上游 i18n / 扩展本地化骨架 / 就地硬改）、不可翻译项清单、中文客户端字符串匹配坑点、存量配置迁移、打包发布。当用户要求汉化、中文化某个 Dalamud/卫月插件，或更新既有汉化 fork 时使用。
---

# FFXIV 卫月插件汉化

适用于把英文 Dalamud 插件做成简体中文版 fork（`*-cn`）。本总结提炼自 5 个已完成的汉化仓库（Allagan Tools、Browsingway、ItemVendorLocation、visland、CharacterPanelRefined）。

## 第 0 步：判断策略（按优先级）

先花 5 分钟确认上游的文本管理形式，再决定动手方式：

1. **上游已有 i18n 体系（resx / Crowdin / 语言 JSON）→ 只补中文翻译，零代码改动。**
   - 标志：存在 `*.resx`、`Language.*.resx`、`crowdin.yml`、locales 目录。
   - 做法：新增或补全 `*.zh.resx`（复制中性文件的 key，只翻 value）。强类型 Designer 类只对中性 resx 生成，不用动。
   - 参考：CharacterPanelRefined（补 `*.zh.resx` 漏翻条目即可）。
2. **上游有简陋本地化骨架（如 CheapLoc 单字典）→ 扩展成完整双语字典。**
   - 做法：把所有用户可见字符串收进字典，代码里换成 `Loc.Localize(key, fallback)`，fallback 保留英文原文；按客户端语言选字典。
   - 参考：ItemVendorLocation（`Localization.cs` 内置 English/Chinese 两本字典）。
3. **纯硬编码字符串（最常见）→ 就地 1:1 替换为中文，单独一个 commit。**
   - 做法：直接把 C# 字符串字面量改成中文。汉化收敛为**独立 commit**，不与逻辑改动混合，便于日后对上游 rebase。
   - 参考：Allagan Tools（543 文件单提交）、visland（34 文件单提交）、Browsingway。

## 通用铁律

### 不用管的

- **字体**：国服卫月自带 CJK 字体支持，插件零字体代码。不要加 `GlyphRanges`/`AddFontFromFile`。
- **布局**：中文普遍比英文短，窗口尺寸一般无需调整。
- **游戏数据文本**：物品名/地名/NPC/动作名等通过 Lumina `GetExcelSheet<T>()` 读国服客户端数据，天然是中文。**绝不自建翻译表**。只需处理 Lumina 空值行的硬编码兜底（如枚举→显示名的 switch）。

### 不能翻的

翻错以下任何一项都会破坏功能或生态兼容：

- `InternalName`、程序集名（除非刻意要与原版共存，见下）、IPC 频道名
- 聊天命令名及其参数取值（`/atools`、`on/off/toggle` 等），只翻 `HelpMessage`
- 配置文件的 Key、ImGui 的 `##`/`###` ID 锚点（只翻显示部分，如 `"应用到全部###Pathfind"`）
- 格式占位符 `{0}`、`{0:N0}`、Discord 时间戳 `<t:...>`——可调整语序但不能改占位符本身
- 嵌入的游戏数据 JSON、解析英文攻略文本用的关键词（解析对象不变，解析器就不动）
- Debug 窗口/调试日志可以不翻，省时省力

### 语言检测

国服客户端的 `ClientState.ClientLanguage` 返回枚举值 `4`（Dalamud 原版枚举无此值）。写法：`(ClientLanguage)4 => Chinese`。上游 resx 方案的语言 switch 通常只有 E/F/G/J，国服会落到 default（英文）——必须给映射补 `zh`。

## 最大的坑：对游戏文本做字符串匹配的代码

汉化后真正会坏的不是 UI，而是**所有把游戏内文本当英文来解析/比较的代码点**。换客户端语言后它们全部失效。逐类排查：

- **tooltip/界面文本前缀检测**：如检测 tooltip 是否已含 `"Shop Selling Price"` 前缀，要改为检测中文 `商店贩售`，否则重复添加。
- **按名字解析游戏窗口内容**：如解析英文商店名 `"Oddly Specific Materials Exchange (Carpentry)"` 找收藏品类别，国服是 `交换改良用材料（刻木匠）`——需加中文名→枚举的字典做双分支。
- **等值比较的语序差异**：英文 `"item min collectability of X"` vs 中文 `"物品名 收藏价值 X 以上"`，`==` 要改成 `StartsWith` 之类。
- 排查方法：grep 源码中所有与游戏界面/数据文本相关的英文字符串字面量（ tooltip、shop、window、sheet 读取后的 `.Name` 比较），逐一确认在中文客户端下的行为。

## 反向利用：锁定英文读数据

国服客户端默认读出中文，但有些场景需要英文名（匹配英文攻略、给英文 bot 发命令）——用 `.WithLanguage(ClientLanguage.English)` 显式指定语言读 Lumina（参考 visland 的 `IExcelRowExtensions.WithLanguage`）。

## 存量用户配置迁移

如果默认配置值是英文字符串（如默认清单名 `"All"`/`"Retainers"`），改代码后老用户配置会失配。教科书做法（Allagan Tools `MigrationManagerService`）：配置版本号 +1，用 `(旧英文名, 类型)` 二元组精确匹配替换成中文，避免误伤用户自建的同名项；判断逻辑同时兼容新旧两种值。

## 发布到自建第三方仓库

本工作区结构：`sources/*-cn` 为汉化 fork，`dalamud-plugins/` 为订阅仓库（`repo.json` + `plugins/<InternalName>/*.zip` + `icon.png`），经 jsDelivr CDN 分发。

1. 构建 Release 包（Dalamud.NET.Sdk + DalamudPackager 自动产出 zip）。resx 方案注意 zip 里必须含 `zh/*.resources.dll` 卫星程序集，缺了会静默回退英文。
2. **版本号多处必须一致**：csproj 的 `Version/AssemblyVersion/FileVersion`、插件清单 `*.json`、`pluginmaster.json`、zip 内 AssemblyVersion。不一致是真实踩过的坑。
3. 插件清单改动：`Name` → "XX 汉化版"，Punchline/Description/Tags 中文化，`Author` 追加 `2488652el(汉化)`；下载地址走镜像（jsDelivr / gh 代理）解决国内 GitHub 访问。
4. 可选——与原版共存：改 `InternalName`+`AssemblyName`（如 `visland-cn`），汉化版可与原版并存且配置独立；不改则与原版互斥。

## 验收清单

- [ ] 游戏内打开每个窗口肉眼过一遍，搜源码 `\p{Han}` 确认覆盖、搜残留英文 UI 串
- [ ] 触发所有聊天命令和错误提示路径
- [ ] 重点实测"字符串匹配游戏文本"的功能点（tooltip 改写、菜单高亮等）
- [ ] 老配置升级路径（若改了默认配置值）
- [ ] zip 内 manifest 版本号 = csproj = pluginmaster.json
- [ ] 订阅地址可在国内网络直接访问

## 维护提示

- 汉化独立 commit 的 fork，上游更新时 `git rebase`/cherry-pick 重放汉化提交；历史压成单提交的（如 Browsingway）只能重新 diff 再翻。
- 上游有 Crowdin/resx 的，优先向上游贡献翻译而不是 fork。
- 建一份术语表（雇员/部队/金币/市场布告板/理符任务/收藏品…），几千处翻译靠它保一致性。
