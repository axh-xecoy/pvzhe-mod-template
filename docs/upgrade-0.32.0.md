# 0.32.0 模板适配记录

日期：2026-11-07。模板起点 `1bac125`，工作树在迁移前无未提交修改；旧默认发行 0.31.0。目标是玩家提供的 Windows C# 0.32.0 发行目录，另用维护者提供的游戏项目源码 main `5f7f09d5c2`（2026-11-06）辅助机制分析。源码未被修改或编译，引用、资源与运行结论均来自发行版。源码 `project.godot` 的版本字段仍为 0.26.0.0，不用于推断发行版本。

## 目标与输入

| 项目 | 0.31.0（旧） | 0.32.0（新） |
| --- | --- | --- |
| PCK SHA-256 | `1f234c9d7954cbbb1cd3f21691d764c8ea9533477ad87818cf4e6a68d7f70f2c` | `ed5850cd3634f231c332fb0e23181f5cae33a4c65cbc911a03df96ce2bfd0528` |
| PlantsVsZombies.dll | `820e9e8f91f58062fcfc7548bdb9e5588ec8f822f5ddc71f4b4d83b09a912d95` | `bcb0c7857ca645651df805e8fd5e442c5055ef1fdeac8f5f44e15766b8e9002c` |
| GodotSharp.dll | `c09b4a1be6a2d56ffbd4992ee6e5ef0cf406d49b99620da5c462da261c7f787e` | 同左（未变） |
| 引擎 / PCK / 条目 | 4.7.0 / v4 / 28,089 | 4.7.0 / v4 / 28,386 |
| 清单 schema / Mod API | v3 / v3 | v3 / v3（入口三方法不变） |
| `project.binary` 版本 | 0.31.0.0 | 0.32.0.0 |

源码 main 与 0.31 基线提交 `28909c0ac9` 之间共 35 个提交，其中包含“0.32 第一批内容”与 `main`→`0.32` 合并。PCK 条目差异：新增 643、删除 346、内容变化 1,711；变化最多的区域是 `.godot/exported/gameplay`（1,460），其次是生存、投射物与关卡配置。详细注册键与资源清单见[0.32增量入口](wiki/updates/0.32.0.md)。

## 影响与改动

| 能力/机制 | 旧证据 | 新证据 | 影响与修改位置 | 验证结果 |
| --- | --- | --- | --- | --- |
| 发行身份与兼容策略 | 0.31 摘要 | 新 PCK/DLL 摘要，引擎与 GodotSharp 未变 | `game-compatibility.json` 新增 0.32 条目，保留 0.31/0.30 声明 | 实际 `prepare` 通过并装入 `.game/current`，0.31 进入 `.game/old` |
| 清单 schema、入口契约 | v3 三方法 | 反射报告与 0.31 逐字节相同 | 无需修改 | `prepare` 反射 + `build` 入口检查 + 真实加载 |
| 公开方法签名 | 0.31 反射 | `GetCampFriendly`、`BungiSpawn`、`CharacterDestroy` 增加带默认值的参数；新增 `GetPacketSlotNum`、`ResolveSlotCount`；`DragMenuSelectItem` 增加生命周期方法 | 模板 C# 未调用这些方法；Wiki 与升级资料记录“旧二进制需重编译” | 四个示例工程对新引用重新编译通过；Hello World 与 FeatureLab 实际运行 |
| 种植音效与预放置 | 无 | 新能力键 `plant-placement-audio-v1`；`plantAudio`/`plantWaterAudio` 字段；`TryPreparePreSpawn` | 记录到[0.32增量](wiki/updates/0.32.0.md)；未改模板代码 | 发行 DLL 字面量/字段与源码静态核对；未运行音效与读档分支 |
| 胜利结算等待 | 单一 `HasPendingBattleOperations` | 新增 `HasPendingVictoryOperations` 分支 | 记录语义变化，不修改示例 | FeatureLab 真实关卡胜利链通过 |
| 载入视图生命周期 | 实例化直接使用 `provider.Scene` | `GC.KeepAlive(scene)` 防重入回收 | 无模板改动；记录到增量 Wiki | 载入场景摘要与 0.31 相同；实际启动通过 |
| 资源路径 | 7 个 `res://` 字面量 | 5 个具体资源与 1 个仍有 18 条目的目录前缀存在，1 个文档占位；删除项 `WitchPoisonBottle` 不在模板引用内 | 无代码改动 | 索引比对 + 运行时加载（关卡纹理、角色场景） |
| 复用目录 | 6 组件、28 类型 | 全部仍存在；3 处方法签名变化、若干新增方法 | `.work/reuse-catalog.json` 重新生成，文档不复制签名 | `catalog.py` 退出 0 |
| UI 组件与挂载 | 按钮/提示/选择/输入/弹窗/主题与 `TowerDefenseControlNew` 场景 | 上述场景与主题摘要与 0.31 逐字节相同 | 无需修改；Hello World 横幅与 FeatureLab HUD 沿用原挂载点 | 两个示例真实运行截图与标记 |
| 攻击预设 | verifiedReleases 含 0.30/0.31 | 预设角色场景、AdobeAnimate 资源与玉米投手组件集摘要与 0.31 一致；0.32 重新采集弹速 300、配置间隔 1.5 秒、采样平均约 1.383 秒、普通僵尸受击矩形 (-18,-37,44,70) | `tools/attack_web/origins.json` 新增 0.32 条目并注明依据；起点几何未改 | 真实捕获 + `attack_recipe new-session` 通过；未在浏览器确认方案 |
| 失焦暂停设置 | 0.31 静态核验 | `Asset/Config/Save/ConfigInit.json` 摘要未变 | 沿用 0.31 说明 | 未重跑失焦行为 |
| 新内容/注册键 | — | 14 个植物键、2 个僵尸键、生存 `PresentNG` 组、6 个新投射物配置、冰/毁灭阳光与图鉴页 | 记录在增量 Wiki；默认模板不加入 | 仅注册表/资源索引事实，未逐项运行 |
| Wiki 组织 | 0.31 增量入口 | 新 `updates/0.32.0.md`，沿用 0.30 主题正文 | `docs/wiki/README.md`、`README.md`、`docs/compatibility.md`、`docs/game-environment.md` | 链接与格式检查 |

## 本轮验证

| 检查 | 结果及覆盖 |
| --- | --- |
| 工具测试 | `python -m unittest discover -s tests -v`：187 项通过（与 0.31 轮同套） |
| 最小模板 | 0.32 `prepare` → `catalog` → `build` → `check` 通过；包内仍只有 `mod.json` 与 `Runtime/ModAssembly.dll`，包摘要 `7a7cc2a8…436583` |
| Hello World 实际运行 | 隔离用户下真实启动：`package applied`、`HELLO_INIT/READY/UI` 三标记、进程退出 0；截图中主菜单左上角可见 `Hello World! · PVZHE Mod`，左下角版本显示 0.32.0.0 |
| FeatureLab | 12 个场景标记通过：UI/弹窗/设置、Mod 关卡进入、HUD 击杀计数、简单规则、真实僵尸死亡与胜利；第二个真实进程读取 bool=false、int=75，`settingsRestartVerified=true`；包摘要 `f59ca1ba…005768` |
| 示例工程编译 | FeatureLab、DragLayout、SuperGatling、AttackLab 对新引用重新编译，0 警告 0 错误 |
| 资源引用 | 7 个 `res://` 字面量逐个对照当前索引；无被删除的完整引用 |
| 干净玩家流程 | 通过：272 个模板文件 + 0.32 发行目录即可完成 prepare/build/check/extract/真实运行 |
| Wiki/文档 | 新增 0.32 增量与升级记录，更新当前基线入口；0.30/0.31 历史实测标签保持原范围 |

真实运行仍沿用此前记录的宿主退出阶段“冻结扩展点清理需要重启”警告（发生在 `HELLO_SHUTDOWN` 之后），未将其包装为无警告运行；`ObjectDB instance was leaked` 一类退出诊断的归因仍未确认。

## 干净玩家流程

将参与版本控制的模板文件（`git ls-files -co --exclude-standard`，272 个文件）复制到 `.work/migrations/20261107-source-5f7f09d5c2/clean-player`，排除 `.game`、`.work`、`dist`、`bin`、`obj`、`__pycache__` 与 `.git`，不提供源码目录；只给出 0.32 发行目录，按 README 执行：

| 步骤 | 结果 |
| --- | --- |
| `prepare --game-dir <0.32 发行目录>` | 通过，自行建立 `current` 槽与资源缓存，28,386 条目 |
| `build` / `check` | 通过；包内仍只有 `mod.json` 与 `Runtime/ModAssembly.dll`，本副本包摘要 `70f93b89…83bbda`（DLL 随构建而非逐字节可复现，身份按各副本自己的摘要） |
| `extract --path Asset/Config/Save/ConfigInit.json` | MD5 校验通过 |
| `runtime_test.py` | 通过：独立用户下真实启动，三个 `HELLO_*` 标记与包/DLL 身份一致，截图可见横幅 |

干净副本没有读取源码、没有使用 Godot 编辑器，也未把机器路径或宿主内容带入产物；`prepare` 之后该副本自带的 `.game`/`.work` 是本地状态，不随模板分发。

测试用户数据：本轮四个隔离目录（`PVZHE Mod Template Test d252af01bcef`、`b796d6b739d6`、`6bcb9d4fbbfc`、`a3637d346a6a`）分别由 Hello World、FeatureLab、干净副本与素材采集的 `runtime_test` 流程创建，路径来自各自 `identity.json`；预览确认其中只有本轮 pmod、Mod 缓存、ModSettings、截图/参考姿态与着色器缓存，无用户存档，且无重解析点，归档证据后逐个清理。登记与结果见同目录 `test-userdata-cleanup.txt`，没有按前缀批量删除，也不触碰玩家日常 profile。

## 未完成与边界

- 攻击编辑器只做到“当前发行可生成并校验会话”：0.32 的采集覆盖单发豌豆/普通僵尸，其他弹药、投手优先索敌、穿透/爆炸与暂停/联机未测；浏览器里的确认/采用流程本轮未运行。
- 新增的 14 个植物、2 个僵尸、`PresentNG` 生存关卡、冰/毁灭阳光与图鉴页只有注册表与资源事实，没有玩法、数值或平衡验收。
- SuperGatling、AttackLab、DragLayout、ModeNavigation 只做了编译（DragLayout 另出包），未在 0.32 上运行；FeatureLab 的双进程持久化只覆盖示例设置键。
- 未验证联机、完整存档兼容、Android、所有布局/金币占位与全部新能力组合；`plant-placement-audio-v1` 的陆地/水上音效、预放置读档还原与拒绝分支仍待专项运行。
- 0.31/0.30 支持仅保留已声明摘要，本轮未用对应发行重新跑一遍旧流程；旧实测记录仍按原日期与版本引用。

## 本地证据

原始对比留在忽略提交的 `.work/migrations/20261107-source-5f7f09d5c2`（`before/` 旧基线、`pck-delta.json`、`source-mod-system.diff`、`analyze-prepare.log`、`prepare-0.32.log`、`tools-tests.log`、`build-*/check-*`、`runtime-*.log`、`attack-capture.log`、`attack-session-0.32*.json`、`clean-player/`）；真实运行轮次在 `.work/runtime/6e4a2f215974`、`.work/runtime-feature-lab/de0cd199f160`，0.32 素材采集在 `.work/attack-preview/capture-78266c47`。报告不含可分发宿主内容，模板与 pmod 均不携带游戏 DLL、贴图或解包结果。
