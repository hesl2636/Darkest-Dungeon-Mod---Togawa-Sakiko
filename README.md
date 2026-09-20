# 丰川祥子 — 暗黑地牢角色 Mod

仿照「惊天大蛆」的 MyGO 系列 mod，为《Darkest Dungeon》补全丰川祥子角色。

- 参考实现：`E:\MyMods\Reference\`（6 个已上架创意工坊的 MyGO / Ave Mujica 角色 mod，目录名即 Workshop ID）
- 目标游戏：Darkest Dungeon 1

**状态：数据框架已建立（职业数据 + 文本 + 校验脚本），美术/动画/音频未接入；核心辅助机制按设计仍待实机验证。**
> 框架口径：英雄暂不进入招募池（`generation` 关闭）；8 个战斗技能的数据均已按参考 mod 实测机制接入（先例与待验证项逐条见 `tools/framework.json` 的 `mechanism` / `gaps`）。本地校验：`node _plan_local/tools/validate.mjs`（静态引用/键位检查，**不等于**游戏内验证）。

---

## 仓库布局

**仓库根目录 = mod 根目录。** `project.xml` 必须位于 mod 文件夹顶层，游戏不接受多嵌套一层目录（`mods/Foo/Foo/project.xml` 是错的）。

```
<repo root>/
├─ project.xml                        工程配置：Title / Language / Tags / ModDataPath / PublishedFileId
├─ preview_icon.png                   创意工坊封面 512×512
├─ modfiles.txt                       文件清单（上传器生成）
├─ heroes/<class>/                    ★ 一个目录一个职业，目录名即 hero class id
│   ├─ <class>.info.darkest           数值：抗性 / 武器 5 级 / 护甲 5 级 / 技能 5 级 / 生成规则
│   ├─ <class>.art.darkest            美术绑定：技能 → 动画名 / fx 名 / 图标名 / 特效偏移
│   ├─ <class>.ability.<one..eight>.png       技能图标（数量 = 战斗技能数）
│   ├─ <class>_guild_header.png       公会界面横幅
│   ├─ <class>_A/
│   │   ├─ <class>_portrait_roster.png        名册头像
│   │   └─ anim/<class>.sprite.<anim>.png     Spine 图集页（贴图）
│   ├─ anim/<class>.sprite.<anim>.atlas + .skel
│   ├─ fx/<class>.sprite.<fx>.atlas + .png + .skel
│   └─ icons_equip/eqp_weapon_{0..4}.png, eqp_armour_{0..4}.png
├─ effects/<class>.effects.darkest    技能效果原子
├─ shared/buffs/<class>.buffs.json    buff 数值定义
├─ upgrades/heroes/<class>.upgrades.json      公会升级树
├─ trinkets/<class>.entries.trinkets.json     专属饰品
├─ raid/camping/
│   ├─ <class>.camping_skills.json    扎营技能（3 条通用 + 4 条专属）
│   └─ skill_icons/camp_skill_<id>.png
├─ panels/icons_equip/trinket/inv_trinket+<trinketId>.png
├─ localization/<class>.string_table.xml      全部文本（english + schinese 两段）
└─ audio/
    ├─ secondary_banks/hero_<class>.bank      FMOD 音频包
    ├─ <class>.heroes.load_order.json
    └─ <class>.heroes.guid_overrides.json     技能 / 扎营事件 GUID 映射
```

可选扩展（按需增加）：`loot/`、`inventory/`、`monsters/`、`shared/party_name/`、`shared/trait/`。

`_plan_local/`（概念图、调研、设计文档、ADR）为本地资料，已被 `.gitignore` 屏蔽，**不属于 mod 内容**；部署到 `mods/` 时必须排除，否则会被工坊上传器一并上传。

---

## 素材规格（实测自参考实现）

| 素材 | 规格 | 数量 | 路径 |
|---|---|---|---|
| 技能图标 | 72×72 RGBA | = 战斗技能数 | `heroes/<class>/<class>.ability.<name>.png` |
| 武器 / 护甲图标 | 72×144 RGBA | 10 | `heroes/<class>/icons_equip/eqp_{weapon,armour}_{0..4}.png` |
| 饰品图标 | 72×144 RGBA | 5 | `panels/icons_equip/trinket/inv_trinket+<id>.png` |
| 扎营技能图标 | 72×72 RGBA | 4 | `raid/camping/skill_icons/camp_skill_<id>.png` |
| 名册头像 | 85×85 | 1 | `heroes/<class>/<class>_A/<class>_portrait_roster.png` |
| 公会横幅 | 715×630 RGBA | 1 | `heroes/<class>/<class>_guild_header.png` |
| 创意工坊封面 | 512×512 | 1 | `preview_icon.png` |
| 角色图集页 | ≤4096×4096，**非 2 的幂亦可**（导出时 Power of two 关闭） | 每组动画 1 张 | `heroes/<class>/<class>_A/anim/` |
| 角色骨骼 | `.atlas` + `.skel` | 每组动画 1 组 | `heroes/<class>/anim/` |
| 特效骨骼 | `.atlas` + `.png` + `.skel` | 6–11 组 | `heroes/<class>/fx/` |
| 音效包 | FMOD `.bank` | 1–2 | `audio/secondary_banks/hero_<class>.bank` |

### 动画清单

必备 8 组：`idle` `combat` `walk` `defend` `camp` `investigate` `heroic` `afflicted`
每个攻击技能 1 组：`attack_<skill>`；可选 `riposte` `guard` `transform` 及第二形态整套。

---

## 硬约束

- **Spine 版本必须是 2.1.27，且必须导出二进制 `.skel`**。`Darkest.exe` 内置 `SpineDataBinaryLoader`，JSON 导出会被显式拒绝（内置错误串 `shouldn't be using spine json files!`）。新版 Spine 导出的 `.skel` 同样无法读取。
- `.atlas` 首行只写裸文件名，引擎按文件名检索；参考实现统一把图集页放在 `<class>_A/anim/`。
- hero class id = `heroes/<class>/` 的目录名与文件名，`info.darkest` 内没有 class 字段；所有引用（饰品 `hero_class_requirements`、buff `spawn_target_actor_base_class_id`、文本 `hero_name_<class>`）都用这个名字对接。
- `info.darkest` 的 `id_index: .index N` 必须全局唯一。参考实现已占用：`3090 3091 3093 3094 4090 4092`。
- 本地化键必须齐全，缺一个就会在界面显示空白键名。参考实现的键位见下节。

### `localization/<class>.string_table.xml` 必需键

```
hero_name_<class> / hero_class_name_<class>
upgrade_tree_name_<class>.<skillId> / upgrade_tree_name_<class>.weapon / .armour
combat_skill_name_<class>_<skillId> / combat_skill_name_<class>_move
camping_skill_name_<id> ×4        str_bark_<id> ×4
<weapon_0..4> <armour_0..4>       装备名
str_inventory_title_trinket<trinketId> ×5
action_verbose_body_{camping_trainer,guild,blacksmith}_<class> ×3
<class>+str_*                     台词，约占全部文本 3/4
<class>+trigger_curio_affliction_* / <class>+trigger_curio_quirk_* ×26
effect_tooltip_* / buff_stat_tooltip_<stat_type>_<sub_type>
```

---

## 开发与部署

1. **本地开发**：仓库根目录即 mod 目录。部署时用目录链接，避免复制：
   ```
   mklink /J "<Steam>\steamapps\common\DarkestDungeon\mods\Sakiko" "E:\MyMods\Darkest-Dungeon-Mod---Togawa-Sakiko"
   ```
   随后在游戏存档选择界面点锤子图标启用 mod。
   > ⚠️ 目录链接会把 `_plan_local/` 一起暴露给游戏目录。**上创意工坊前必须先把 `_plan_local/` 移出该目录**（或改为「只拷贝 mod 文件」的部署方式），否则概念图与调研文档会被一并上传。
   >
   > 本机当前未检测到 Darkest Dungeon 安装目录（`G:\SteamLibrary\...` 不存在），部署前需先确认游戏路径。
2. **发布**：官方 Steam Workshop Uploader 读取 `project.xml`，自动编译 `localization/*.string_table.xml` → `<PublishedFileId>_<lang>.loc2`，并重写 `modfiles.txt`。

### 生成物

`modfiles.txt`、`localization/*.loc2` 由上传器生成，不要手写。默认纳入版本管理以便直接部署；若只把仓库当源码，可在 `.gitignore` 中取消对应两行注释。

---

## Git

- `origin` → https://github.com/hesl2636/Darkest-Dungeon-Mod---Togawa-Sakiko
- 行尾：仓库内统一 LF（见 `.gitattributes`），二进制资源禁用任何转换。
- 提交者身份写在仓库本地配置中（`hesl2636 <hesl2636@users.noreply.github.com>`），未改动全局 git 配置。
