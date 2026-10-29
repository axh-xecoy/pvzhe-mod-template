---
name: pvzhe-mod-verify
description: 安装、启用并实际运行验证 PVZHE pmod，处理独立测试环境的首次启动提示，收集包身份、日志和界面证据；适用于功能验收与发布前检查。
---

# 安装与验证

先按 [Wiki 索引](../../../docs/wiki/README.md) 核对发行版本与摘要，再按需读取 [startup](../../../docs/wiki/0.30.0/startup.md)、[battle-testing](../../../docs/wiki/0.30.0/battle-testing.md)、[build-validation](../../../docs/wiki/0.30.0/build-validation.md)。匹配基线时沿用已确认的操作路径；仅对缺项、差异或失败点补充分析，不从零遍历游戏。静态确认和待验证内容不能冒充运行验收。

输入：已构建 pmod、匹配的发布游戏、需求验收项与测试环境。输出：当前产物的加载、功能、界面证据和未验证项。

用户准备分享/发布时，先按 [身份与发布规范](../pvzhe-mod-develop/references/identity-and-release.md) 确认名称、作者与版本，重建最终包并执行 check --release；已安装 ID 冲突只在确认是同一 Mod 升级后允许，不把发布字段检查当作实际功能验收。

自动化对局采集前，按[测试会话准备](references/test-session.md)关闭隔离环境的失焦自动暂停并核实生效，保留手动/模态暂停。启动前合并可共用进程的采集项；失败先在当前会话诊断，重启须有具体原因，不靠重复启动和抢焦点推进测试。

1. 先按 [自定义功能验收](references/feature-acceptance.md) 推导成功与不应发生的行为；自定义验收或重验先读 [验证执行策略](references/verification-strategy.md)，明确本轮条件、可证伪断言及受影响范围，再 build/check。游戏采集、证据判定与报告分开，报告修改先离线执行，不重启整套。内置运行工具只测试 Hello World 或 FeatureLab；自定义功能需另设对应验收，不因示例通过而宣称需求完成。命令与证据范围见 [内置示例验证](references/example-scenarios.md)，仅测试这些示例时读取。
2. 优先在独立用户目录验证，记录实际路径与测试初始状态。最终生产包也能在隔离环境运行；人工试用不等于安装到日常存档。首次启动、用户名弹窗或复用测试存档时读 [首次启动与存档前提](references/first-run.md)。环境准备与业务断言分别记录，测试驱动不得变成正常 Mod 的启动行为；最终包另验不带测试参数的普通启动，清单声明内容不能只在测试分支注册。
3. 常规安装先退出游戏，用 `python tools/mod.py install --mods-dir '<确认的 user://Mods>'` 安装当前构建；示例可加 `--example`。在 Mod 管理启用并重启，核对安装包与构建包摘要。安装目录以宿主实际 user:// 为准。
4. 本轮产物与证据独立保存；后台验收期间不重新构建或覆盖共享输入，检查器失败不沿用旧报告。使用本次启动日志核对 package applied、入口及功能证据、当前 DLL 摘要，检查最后一次初始化之后的诊断。内置 `mod.py verify` 只认识示例标记；自定义 Mod 需对应检查器。日志证明回调执行，截图和交互检查证明实际可见、可用；程序化事件不等于真实鼠标、音效或自然通关验收。
5. 按需求检查再次触发、切场景、禁用重启与存档恢复。只报告实际覆盖项；启动失败、布局异常、资源/身份错误及退出诊断按需读 [运行排障](references/runtime-troubleshooting.md)。保留日志、退出码和失败证据，不将未知诊断过滤为成功。涉及可定位浮动UI时按ui-drag的[玩家试用交接](../pvzhe-ui-drag/references/player-layout-review.md)提供调整机会；最终图按[交付证据](references/feature-acceptance.md#最终交付证据)检查，不以自动验收退出代替交接。

遇到超时、崩溃或不兼容，先定位失败阶段；重跑须说明修复、条件变化或新增观测；同一断点连续两轮未定位先停完整回归、做定向诊断，按执行策略继续推进。证据未变化时不反复重试，不靠修改玩家原存档绕过。实测记录放入验证报告；可复用的机制与解决步骤归入相关 Skill 的参考资料，注明发布版本及验证边界。

磁盘与隔离：按[本地环境](../../../docs/game-environment.md)复用.game/current/runtime，串行持锁；新轮次只新增.work证据及测试PCK补丁，不整包复制游戏。整理历史副本先runtime_storage preview；只清理摘要可重建的游戏文件，失败日志/截图/Mod仍保留，不触碰日常存档。
