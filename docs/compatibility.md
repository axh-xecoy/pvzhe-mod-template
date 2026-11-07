# 发布基线与证据

当前默认发行已升级到Windows C# 0.32.0，PCK v4、28,386条目，Godot仍为4.7.0，Mod schema/API仍为v3。准确摘要、语义差异及新验收范围见[0.32增量入口](wiki/updates/0.32.0.md)和[升级记录](upgrade-0.32.0.md)。game-compatibility.json同时保留0.31与0.30摘要；0.31的[增量说明](wiki/updates/0.31.0.md)与原0.30功能实测记录按原范围保留，不改名为新版结果。以下保留原始0.30基线与0.31基线摘要，供差异与历史验收引用。

基线为 Windows C# 发布版 0.30.0（`project.binary` 标记 0.30.0.0），PCK 引擎标记 4.7.0、格式 v4、27,426 条目；真实运行显示 `Godot Engine v4.7.stable.mono.official.5b4e0cb0f`。发布运行时为 net9.0，随包运行时 9.0.17。开发实测 Python 3.12、.NET SDK 9.0.316。

发布程序集反射确认：`XWModManifest.CurrentSchemaVersion = 3`、`XWModRuntimeApi.CurrentVersion = 3`，入口契约包含 `Initialize(XWModRuntimeContext)`、`OnAllModsLoaded()`、`Shutdown()`。本示例使用受支持的 API v1 UI 挂载点，清单 schema v3；API 版本数字不等于游戏版本或 Godot 版本。

本机分析期间内部源码用于定位机制，但交付的准备、编译、解包与运行脚本均只接受发布包及本模板，不读取内部源码。运行时编译接口来自发布 DLL；核心协议以发布实测为准。源码配置的版本字段与发布包不同，不能用内部源码版本替代发布证据。

`.work/release.json` 保留本机发布 PCK 与引用 DLL SHA-256、格式与条目数；`.work/api.json` 保留入口与常用服务接口；`dist/build-info.json` 保留最终包摘要与允许名单。原始资源、PCK、DLL 和包含机器路径的本地报告均不分发。

PCK 二进制解析依据 [Godot 官方 file_access_pack.cpp](https://github.com/godotengine/godot/blob/master/core/io/file_access_pack.cpp) 的头部、目录偏移、文件偏移和摘要字段。工具独立实现解析，仅支持未加密独立包；原始提取不意味着逆向恢复完整编辑器工程。

## 验证范围

准备与构建必须在仅有发布包的条件下完成，产物检查应证实仅含 `mod.json` 与自己的 DLL。实际运行验证还需检查宿主 applied 状态、三个 `HELLO_*` 标记及主菜单显示；肉眼布局检查与日志检查分别报告。自动验证记录保存在 `dist/verification.json`，最终实测记录见 [validation.md](validation.md)。

实测发现发布 EXE 禁止 `--main-pack`，且本机 headless 启动在宿主加载阶段原生崩溃；`runtime_test.py` 采用独立发布副本和 OpenGL 图形运行。测试修改副本用户目录配置并加入专用 GDScript 驱动，触发已有开始按钮、抓取游戏渲染画面，没有替换宿主 C# 代码或 Mod 内容。若发布包改变格式、接口或运行方式，停止沿用本基线，先重新分析、更新证据与运行测试。
