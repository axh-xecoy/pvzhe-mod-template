# 安卓真机安装与验证（无 root）

用途：把已构建的 `.pmod` 装进安卓真机、改配置、取日志、复现问题。**不需要 root，也不需要游戏源码**。

证据状态：以下路径与命令在 **PVZHE 安卓包 0.32.0.0 + Android 16（ELP-AN00，无线 adb）** 上实测；标「待验证」的条目尚未实测，不要当结论引用。PC 侧的安装与验证见 [README 的安装与运行](../../../../README.md#安装与运行)。

## 1. 先确认 `user://` 在安卓上的真实位置

| 环境 | Mods 目录 |
| --- | --- |
| Windows | `%APPDATA%\Godot\app_userdata\植物大战僵尸杂交版\Mods` |
| Android | `/data/user/0/<包名>/files/Mods`（= `/data/data/<包名>/files/Mods`） |

实测（0.32.0.0，包名 `xvwugame.PlantsVsZombies`）：`files/` 下同时存在 `Csharp/`（存档）、`ModSettings/`、`Mods/`、`ModsCache/`、`logs/`。

⚠️ `/sdcard/Android/data/<包名>/files/` **可能是空的**，不要把它当成 `user://`。宿主实际 `user://` 以「能读到 `Csharp/save.res` 与 `Mods/enabled_mods.json` 的那个目录」为准。

## 2. 三条通路，按代价从低到高

| 通路 | 前提 | 能做什么 | 主要代价 |
| --- | --- | --- | --- |
| **A. `adb run-as`**（最省事） | 目标包带 `DEBUGGABLE`（需自己改包，见 C） | 读写整个 data 目录、取日志、批量推包 | 需要电脑 + adb；仅限本包；包必须是改签名的 |
| **B. MT「注入文件提供器」** | 能改 APK 并安装 | 手机上直接用文件管理器读写 data 目录 | 需注入并重装；**目标进程必须运行**；APK 自校验会失效 |
| **C. 给 APK 加 `android:debuggable="true"`** | 官方包默认不带该标志 | 打开 `run-as`/调试能力（A 的前提） | **必须重签名**，同包名不同签名不能覆盖安装 |

### A. `adb run-as`

`run-as` 需要目标包带 `DEBUGGABLE` 标志。**官方发行包默认不带**；实际做法是自己改包（见通路 C）后再装，或者直接用别人改好的 debuggable 包。本仓验证者就是把官方 APK 加上 `android:debuggable="true"` 重新签名安装后，`adb run-as` 才可用的。

先确认标志位：

```powershell
$adb = "C:\...\platform-tools\adb.exe"
$serial = "192.168.1.124:5555"
& $adb -s $serial shell "dumpsys package xvwugame.PlantsVsZombies" | Select-String 'flags='
```

带 `DEBUGGABLE` 时输出形如 `flags=[ DEBUGGABLE HAS_CODE ALLOW_CLEAR_USER_DATA ALLOW_BACKUP ]`，此时 `run-as` 直接可用（设备无需 root；`id` 是 `shell` 也能降权到应用 uid）。反过来说，**看到 `DEBUGGABLE` 只说明"这台机器上装的是改过的包"，不能据此认为发行包本来就带**。

```powershell
# 列目录
& $adb -s $serial shell "run-as xvwugame.PlantsVsZombies ls -la files/Mods/"

# 读某个配置文件
& $adb -s $serial shell "run-as xvwugame.PlantsVsZombies cat files/Mods/enabled_mods.json"

# 取日志（存档同理，路径换 files/Csharp/save.res）
& $adb -s $serial shell "run-as xvwugame.PlantsVsZombies cat files/logs/godot.log" > godot.log
```

`run-as` 的 cwd 是 `/data/user/0/<包名>`，所以远端路径写 `files/Mods/...`。进程不运行也能读写。

### B. MT 管理器「注入文件提供器」（DocumentsProvider）

参考 MT 官方说明：<https://mt.cc/guide/reverse/inject-documents-provider.html>。原理是往 APK 里注入一个专用 `DocumentsProvider`，把 data 目录通过系统文件接口导出，再配合 MT 的「本地存储」实现**无 root 访问与修改 data 目录**。

流程（以官方文档为准）：

1. MT 里找到目标 APK → 「功能」→「注入文件提供器」；
2. 得到 `xxx_dp.apk`，安装并**打开一次**；
3. 回 MT → 侧拉栏 → 右上角「添加本地存储」→ 侧拉栏中选中目标 App → 底部「选择」；
4. 之后从 MT 进入该本地存储，即可浏览/修改 data 目录。

导出目录对照：`android_data` = `/sdcard/Android/data/<包名>`，`android_obb` = `/sdcard/Android/obb/<包名>`，`data` = `/data/data/<包名>`，`user_de_data` = `/data/user_de/0/<包名>`（MT ≥ 2.14.3）。

支持浏览、新建/删除/改名/移动、查看与修改权限和时间、创建软链接；不支持改所有者。第三方文件管理器也能访问，但软链接判定、权限/属主查看等 MT 定制能力不可用。

局限（官方明示）：**APK 有自校验时很可能失效**；**浏览目标 App 文件时其进程必须正在运行**，否则取不到内容（MT 会提示并支持一键启动）。

适用场景：手边没有电脑，想在手机上现场贴 `.pmod`、改 `enabled_mods.json`、导出日志。**本模板未实测该通路**（见 §5）。

### C. 给官方 APK 加 `android:debuggable="true"`（通 A 的实际前提）

要点：`apktool d` 解包 → 在 `AndroidManifest.xml` 的 `<application>` 上加 `android:debuggable="true"` → `apktool b` 回编 → **用别的签名重签** → 安装。

维护者实操结论（实测）：改完重签的包能正常安装运行，`dumpsys package` 显示 `DEBUGGABLE`，`adb run-as` 随即可用（本仓 §3 的推送/备份流程就是在这样的包上跑通的）。

必须记住的代价：

- 同包名 + 不同签名**不能覆盖安装**：得先卸载官方包 ⇒ **`/data` 下的本地存档（`files/Csharp/`）、已装 Mod、Mod 配置一起被清除**。第一次这样装之前先确认没有要保的进度，之后每次更新同理；
- 游戏每次更新都要重做一遍（新包 → 再改 → 再重签 → 再卸载重装 → 再回灌 Mod 与存档）；
- 若发行包带完整性/自校验，重签名可能直接触发拦截（**待验证**：本作上述流程未遇到拦截）。

## 3. 用 adb 推送 pmod 的正确姿势（踩坑记录，均实测）

1. **不要用 `adb shell ... > 文件` 传二进制。**
   实测：108,940 B 的 pmod 只写进 **278 B**（`adb shell` 的管道会按行处理/截断）。正确做法是走 base64 文本：

   ```powershell
   $local = "...\FpsDisplay.pmod"
   $b64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes($local))
   [IO.File]::WriteAllText("C:\tmp\x.b64", $b64, [Text.Encoding]::ASCII)
   $cmd = "run-as xvwugame.PlantsVsZombies sh -c 'base64 -d > files/Mods/_p01.pmod'"
   Start-Process -FilePath $adb -ArgumentList @('-s',$serial,'shell',$cmd) -RedirectStandardInput "C:\tmp\x.b64" -NoNewWindow -Wait
   ```

2. **文件名含空格或中文时，绝不要拼进远端命令。**
   外层 shell 会在空格处拆词，实测产生一堆 0 字节的残文件（`FPS`、`死亡界面`、`三倍加速`…）。正确处理：先推成 ASCII 临时名，再把 `mv` 放进一个**设备端脚本**（同样用 base64 推上去的 `.sh`）里执行，脚本内容为 UTF-8。

3. **`tar` 的相对路径**：`run-as ... sh -c 'tar -cf - files/Mods'` 可用；`tar -C files/Mods` 会报 `chdir ... Permission denied`，改用先 `cd` 或写绝对路径之外的相对根。

4. **备份走「设备端 tar + base64」**，不要直接二进制重定向：

   ```powershell
   & $adb -s $serial shell "run-as xvwugame.PlantsVsZombies sh -c 'tar -cf - files/Mods files/ModSettings | base64'" > bak.b64
   # 本地 base64 解码后即为完整 tar（实测 3.2 MB / 78 条目）
   ```

5. **推完必须逐个 `md5sum` 比对**：`run-as <pkg> sh -c 'cd files/Mods && md5sum *.pmod'`，与本地 dist 对表，19/19 全中才算成功。

6. **装完清宿主状态**：删 `files/Mods/mod_boot_journal.json` 与 `mod_last_good.json`，清空 `files/ModsCache/`。否则宿主可能拿旧包摘要做回滚或进安全模式的加载流程。

7. **`enabled_mods.json` 是无 BOM 的 UTF-8**；更新时保留玩家原有启用项，只追加/移除本次变化的 id，不要整份覆盖成模板默认值。

## 4. 真机验收清单

| 步骤 | 成功条件 |
| --- | --- |
| 启用并重启 | `files/Mods/enabled_mods.json` 含目标 id；重启后 `files/logs/godot.log` 出现 `package applied` 与入口初始化行 |
| 看界面 | 目标 UI/功能实际出现；程序化事件不等于真实触摸验收 |
| 取证据 | 日志用 `run-as ... cat files/logs/godot.log`；截图用 `adb exec-out screencap -p > shot.png`（`exec-out` 才不会被 CRLF 破坏） |
| 与 PC 对齐 | 同一份包在 PC 与手机上的 md5 一致，避免"手机上装的是旧包"这种假结论 |
| 复测 | 禁用重启、切场景、退出重进后无残留 |

## 5. 待验证

- MT「注入文件提供器」在本游戏包上的实测（本文只依据官方说明整理流程与风险，未在本 Mod 环境实际注入）；
- 重签名包（通路 C）对**游戏内更新提示/完整性自校验**的影响：本次实操未被拦截，但未做针对性验证；
- 「重签名必须卸载」导致的存档迁移流程：本文只给出「先备份 `/data` 再卸载」的原则，跨签名回灌 `files/Csharp/` 后的进度兼容性未实测；
- 无线 adb 在长时间批量传输下会偶发 `device offline`，重连用 `adb connect <ip>:5555`；大包建议分文件推送并逐个校验。
