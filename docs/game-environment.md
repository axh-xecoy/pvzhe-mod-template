# 本地游戏环境与模板更新

游戏内容只来自玩家提供的发行目录，不进入Git或pmod。.game和.work均被忽略，不能用git add -f分发。源码不是玩家前置条件。

```text
.game/
  current/                 当前版本：EXE、PCK、data_*、release-manifest.json
    resources/             当前发行的索引、按需资源及预览素材
    runtime/               可复用的隔离运行环境，一份可修改测试PCK
  old/                     上一版本，同样包含自己的资源/运行环境
  environment.lock         准备、构建、运行和清理的互斥锁
.work/
  references/              当前编译引用
  release.json api.json    当前准备证据
  game-cache/current.json  指向当前资源缓存的导航记录
  <用例>/<轮次>/            日志、截图、Mod/DLL、驱动、身份与PCK补丁
  template-updates/        模板差异、三方比较基线、本地备份
game-compatibility.json    提交：模板支持的发行摘要与显示版本
template.lock.json         提交：模板上游与来源提交；初始revision=null
```

## 准备、复用和两版本轮换

首次执行`python tools/mod.py prepare --game-dir '<玩家发行目录>'`。工具先检查PCK、EXE、完整data_*文件清单和摘要，并匹配模板支持的PCK/关键DLL/引擎基线。新副本校验通过后才轮换：current变old，新版成为current，原old删除。临时incoming/retired目录只是事务过程，不作为第三个长期版本；发现中断事务时先调查，不自动清空。

以后直接`python tools/mod.py prepare`，复用current，不重新复制游戏；资源按需提取到current/resources。版本名称相同但文件摘要不同仍视为不同发行；不能靠目录名、DLL的1.0.0.0或mtime判定。build同时核验环境、模板兼容声明与引用身份；只把官方模板更新到0.31、仍配0.30时会拒绝混用。

`python tools/game_environment.py status`查看current/old。需要回退且模板仍支持旧发行时，执行`python tools/mod.py prepare --game-dir '.game/old'`，交换两槽并重建引用；不保留第三份。两版淘汰意味着更早的游戏输入不再本地保留，历史报告不被删除，但完整复现可能需玩家再次提供匹配摘要的旧发行。

原始current文件不被测试改写。运行使用current/runtime中的一份可修改PCK，EXE/data_*只初始化一次；每次驱动或隔离用户设置不同，可以重写这一份PCK，不为每轮永久复制整套游戏。所有本项目验证串行持有environment.lock，避免其他任务覆盖正在运行的输入。锁记录PID；异常退出后须先确认进程/游戏已结束再移除遗留锁，不能把锁错误当作无限重试理由。

## 证据与磁盘整理

发行副本与测试用户数据分开整理：下述runtime_storage命令不回收Godot的user://目录。测试目录登记、保留和安全回收按[测试用户数据生命周期](../.agents/skills/pvzhe-mod-verify/references/userdata-lifecycle.md)，路径从环境/运行记录取得；禁止清空Godot用户数据根或按Test前缀批量删除，玩家日常存档、设置和Mod始终排除。

每轮保留日志、截图、退出/业务报告、源码/Mod/DLL摘要以及驱动，不共享可变业务证据。pack-replay.json与test-pack.tail.bin记录原版PCK摘要、测试头部、追加数据及测试PCK摘要，可精确恢复测试包；不将测试包补丁混入发布Mod。已结束旧轮次可整理：

```powershell
python tools/runtime_storage.py preview
python tools/runtime_storage.py clean
python tools/runtime_storage.py restore --runtime '<.work中的轮次runtime目录>'
# 更早基线已被两槽淘汰时，玩家明确提供原发行PCK
python tools/runtime_storage.py restore --runtime '<目录>' --base-pck '<匹配摘要的原PCK>'
```

preview仅报告可回收内容；clean只处理可由当前基线精确重建的PCK及逐文件摘要一致的发行EXE/data_*，保留日志、截图、Mod及业务代码；不直接删除整个.work。不同版本或不符合追加型测试包的文件跳过。工作区游戏进程运行时拒绝清理。恢复校验基线、补丁和完整PCK摘要，并恢复已归档的发行运行时文件；不会启动游戏。恢复后的完整副本可再次clean。旧.work/game-cache保留给历史引用，新准备使用.game内资源缓存，不擅自删除被会话引用的素材。

## 两种更新的职责

| 意图 | Skill | 主要操作 |
| --- | --- | --- |
| 玩家更新自己的Mod项目，获取官方模板的新工具/Skills | pvzhe-project-update | 获取上游差异，保护自己的功能，合并后核对本地游戏基线 |
| 维护者研究尚未支持的新游戏，并发布适配后的模板 | pvzhe-template-upgrade | 对照发行/明确提供的源码研究，更新契约、工具、Wiki和Skills，发行版验收 |

使用者更新先`python tools/project_update.py plan --ref '<官方分支/标签/提交>'`，不替换origin，也不直接pull覆盖源码。工具将上游版本解包到.work/template-updates，输出changed、overlaps和environment，保留本地补丁与未跟踪文件备份。上游暂存文件是待审查输入，审查前不运行其中程序。

AI按base→target的变更做三方合并，避免把玩家源目录整体替换成Hello World；保留Mod身份、业务代码、资源、设置与design/attacks。重叠文件需理解两边意图，未重叠的上游删除也要检查是否已被用户扩展引用。新模板兼容当前.game则prepare直接复用；不兼容则取得受支持发行或选择其他模板版本，不能修改摘要伪装匹配。

template.lock.json初始revision=null，不猜测为玩家当前HEAD。首次更新从Git共同祖先确定基线；无共同历史时补齐原模板提交后用--base指定，不能强行覆盖。合并、构建和受影响功能验收通过后，`python tools/project_update.py record --plan '<plan.json>'`登记新来源；record不执行业务验收，也不自动提交/推送。

维护者分析未知发行可执行`python tools/mod.py prepare --analyze-only --game-dir '<新版发行目录>'`，证据写入独立.work/release-analysis，不替换当前环境与引用，不允许据此正常构建。随后按维护者Skill更新兼容声明并完成真实prepare/验收，不能把未知发行直接加入支持列表绕过研究。

分析阶段提取未知发行资源时显式指定`extract --pck '<新版PCK>' --path '<资源>' --output '<本轮分析目录>/extracted'`；不带--dependencies的独立提取不会误用当前发行的共享缓存。使用该分析目录的索引与报告，不把主.work中的旧报告当成新版结果。

本轮自动化与实际发行验证范围见[验收记录](game-environment-validation.md)。
