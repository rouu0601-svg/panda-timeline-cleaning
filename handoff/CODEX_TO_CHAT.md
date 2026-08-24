# Codex -> ChatGPT handoff

ROUND=HOST_FOREGROUND_LAUNCHER

HANDOFF_FILES_CREATED=YES
FOREGROUND_LAUNCHER_CREATED=YES

EMULATOR_COMMAND=
`D:\\CodexData\\MagicWarriorCompat\\runtime\\emulator\\emulator-arm.exe -verbose -no-audio -no-boot-anim -no-snapshot -no-accel -memory 768 -gpu host -ports 5590,5591 -sysdir D:\\CodexData\\MagicWarriorCompat\\runtime\\system-image -system ...\\system.img -kernel ...\\kernel-qemu -ramdisk ...\\ramdisk.img -datadir D:\\CodexData\\MagicWarriorCompat\\matrix\\gpu_host -data ...\\userdata-qemu.img -cache ...\\cache.img -initdata ...\\userdata.img -partition-size 650 -netdelay 0 -netspeed full`

NO_WINDOW_FLAG_PRESENT=NO
PROCESS_LIFECYCLE_PRESERVED=YES
LOG_PATH=D:\\CodexData\\MagicWarriorCompat\\matrix\\host_foreground\\emulator_console.log

USER_ACTION_REQUIRED=
双击 `D:\\CodexData\\MagicWarrior\\Start_MagicWarrior_HOST_FOREGROUND.bat`，等待最多 60 秒。只观察是否出现 Android Emulator 窗口并进入 Android 桌面；暂时不要点击《魔界勇士》。窗口出现后，下一轮再检查 ADB 和应用级网络隔离。如果 60 秒仍无窗口，只需回复 `HOST_WINDOW=NO`。

FILES_CHANGED=
- 新增 `D:\\CodexData\\MagicWarrior\\Start_MagicWarrior_HOST_FOREGROUND.bat`
- 新增本交接文件和 `CURRENT_STATE.md`
- 未修改 `Start_MagicWarrior.bat`、`Start_MagicWarrior_HOST_MANUAL.bat`、APK、现有 AVD、Windows 网络或防火墙

BLOCKER=
Codex sandbox 没有交互式 Windows 桌面，不能替用户判断 GUI 窗口。此前前台诊断显示 emulator-arm.exe 能启动并持续约 30 秒，没有立即闪退；旧 BAT 的 `start` 会分离进程并丢失自然退出状态。

NEXT_STEP=
等待用户完成一次可见桌面人工测试。只有窗口出现后，才继续 ADB 状态和应用 UID 网络隔离验证。

PUSH_STATUS=SYNCED

---

ROUND=HOST_FOREGROUND_EXIT_CODE_3_DIAGNOSIS

HOST_WINDOW=NO
EMULATOR_EXIT_CODE=3
LOG_PATH=D:\\CodexData\\MagicWarriorCompat\\matrix\\host_foreground\\emulator_console.log

ROOT_CAUSE=
日志末尾明确报错：`This application failed to start because it could not find or load the Qt platform plugin "windows".` 模拟器此前已经读取 system image、userdata、cache、hardware-qemu.ini，完成 GPU host 配置并生成 QEMU 参数；因此退出码 3 的直接原因是 Qt Windows 平台插件缺失或未被 Qt 定位，不是 APK、ADB、网络、镜像或端口问题。

MINIMAL_FIX_SUGGESTION=
只在下一轮获准后处理：确认同一 Emulator 25.2.5 包内的 `lib\\qt\\plugins\\platforms\\qwindows.dll` 存在，并让启动器把 Qt 平台插件目录显式指向该目录（例如 `QT_QPA_PLATFORM_PLUGIN_PATH`）。若文件确实缺失，应从同一官方/原始 Emulator 包恢复匹配文件；本轮未执行复制、修改或启动验证。

ACTION_TAKEN=
仅读取并分析日志；未启动 APK、未 wipe-data、未修改原 APK、网络、防火墙、VMware 或 C 盘，也未更换 Emulator。

NEXT_STEP=
待用户确认后，再做 Qt 插件路径/文件的只读确认；确认后再决定是否制作单独的启动器修复副本。
