# MagicWarrior current state

## Stable facts

- 原 APK 未修改，SHA256：`7977a38cdc22ae0c7f6c384feaba174961b7672c25fd08b1abf754c13e9e8cd6`
- Android 5.0.2 / API 21 / ARM32 可安装并启动。
- 单机流程已确认可创建角色、进入地图和启动首场战斗；服务器研究已停止。
- `gpu=off` 是保底运行方案；存在白屏和彩块问题。
- `gpu=host` 已确认菜单、地图明显改善；完整战斗尚未验证。
- guest 路线、新版 Emulator、音频和服务器均暂缓。

## Launcher status

新增：`D:\CodexData\MagicWarrior\Start_MagicWarrior_HOST_FOREGROUND.bat`

- 使用旧 Emulator 25.2.5 的 `emulator-arm.exe`。
- 使用 API21 ARM32 的现有 host userdata/cache 和 D 盘 system image。
- 使用 `-gpu host`、console/ADB ports `5590,5591`。
- 不使用 `start`；emulator 直接占用用户当前控制台，直到自然退出。
- stdout/stderr 写入 `D:\CodexData\MagicWarriorCompat\matrix\host_foreground\emulator_console.log`。
- 启动前只检查 emulator、system、kernel、ramdisk、userdata、cache 是否存在。
- 不自动安装或启动 APK，不添加网络规则，不修改防火墙，不 wipe-data。

## Current blocker

人工前台测试结果为 `HOST_WINDOW=NO`、`EMULATOR_EXIT_CODE=3`。指定日志显示 emulator 已完成镜像和 QEMU 参数初始化，随后报错：`This application failed to start because it could not find or load the Qt platform plugin "windows".` 当前直接阻塞点是 Qt Windows 平台插件缺失或未被定位。此前的 `start` 分离问题已经由前台启动器解决，不再是本轮根因。

## Minimal fix suggestion (not applied)

下一轮仅先只读确认同一 Emulator 25.2.5 包中的 `lib\\qt\\plugins\\platforms\\qwindows.dll`，以及 Qt 平台插件搜索路径。最小候选是设置 `QT_QPA_PLATFORM_PLUGIN_PATH` 指向该 `platforms` 目录；若 DLL 缺失，再从匹配的原始 Emulator 包恢复。当前没有执行任何修复。

## Qt plugin check

- 目标文件不存在：`D:\\CodexData\\MagicWarriorCompat\\runtime\\emulator\\lib\\qt\\plugins\\platforms\\qwindows.dll`
- 对旧 Emulator 目录的只读搜索未找到可确认的 `qwindows.dll`。
- 现有前台 BAT 的 `QT_QPA_PLATFORM_PLUGIN_PATH` 未设置；仅扩展了普通 `PATH`。
- 因 DLL 缺失，本轮没有创建或运行 QT 路径测试启动器，窗口结果和退出码均为 `NOT_TESTED` / `NOT_RUN`。
- 官方 Android Qt 预编译目录显示标准插件路径为 `windows-x86/plugins/platforms/qwindows.dll`；Emulator 25.2.5 官方归档已下载到独立目录，未替换原 runtime。

## Official 25.2.5 package and qtfixed runtime

- 官方来源：`https://dl.google.com/android/repository/tools_r25.2.5-windows.zip`
- 下载 SHA256：`da1a0bd9bb358cb52a8fc0a553a060428efe11151e69b9ea7a5cbacb27cf1c7c`
- 官方 x86 qwindows：`D:\\CodexData\\MagicWarriorCompat\\official_sdk_tools_25_2_5\\tools\\lib\\qt\\plugins\\platforms\\qwindows.dll`
- 官方包同时包含 `tools\\lib64\\qt\\plugins\\platforms\\qwindows.dll`；当前 `emulator-arm.exe` 与官方 `tools\\emulator-arm.exe` SHA256 相同，故使用 x86 `tools\\lib` 树。
- 当前 runtime 的 `emulator-arm.exe`、Qt5Core、Qt5Gui、Qt5Widgets 与官方 x86 对应文件 SHA256 相同；缺失的是整个 Qt plugins 子树，而不是核心 Qt DLL。
- 已构建 `D:\\CodexData\\MagicWarriorCompat\\runtime_qtfix`：完整复制旧 runtime 后，只增加官方 x86 Qt plugins 子树（13 文件，约 3.23 MB）。
- 已创建 `D:\\CodexData\\MagicWarrior\\Start_MagicWarrior_HOST_QTFIX.bat`：仅引用 `runtime_qtfix`、设置 `QT_QPA_PLATFORM_PLUGIN_PATH`、使用 `-gpu host`，不自动安装/启动 APK。
- 原 runtime 未修改，原 userdata/cache 未修改。

## Next candidate action

下一步用户在正常 Windows 桌面运行 QTFIX 启动器，只观察窗口和退出状态；暂不启动 APK，不检查 ADB、网络、音频或 guest。

## GitHub handoff sync

本轮 handoff 更新待提交并推送。
