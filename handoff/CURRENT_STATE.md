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

## Next candidate action

下一步先确认 Qt 平台插件文件和搜索路径；确认后再决定是否制作单独的启动器修复副本。暂不启动 APK，不检查 ADB、网络、音频或 guest。

## GitHub handoff sync

本轮 handoff 更新待提交并推送。
