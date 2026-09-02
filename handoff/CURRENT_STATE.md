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

## Live QTFIX black-screen diagnosis

- 用户人工结果：`QTFIX_WINDOW=YES`、`ANDROID_DESKTOP=NO`、`SCREEN=BLACK_PERSISTENT`；本轮将该实例视为仍在运行。
- `emulator-arm.exe` PID 15644 仍为 Running；未发现独立 qemu-system 进程（旧版 QEMU 嵌入 emulator-arm.exe）。约 6 秒只读 CPU 累计时间从 `0:13:00` 增至 `0:13:06`，因此进程仍有活动。
- `D:\CodexData\MagicWarriorCompat\matrix\host_qtfix\emulator_console.log` 最后阶段是 `Starting QEMU main loop`、`Initializing hardware OpenGLES emulation support`、监听 console `5590`/ADB `5591`、Serial `emulator-5590`；没有 boot completed、桌面或应用日志。
- 日志没有 EGL/GLES/FBO/native crash 错误；只有启动时 `can't connect to ADB server: connection refused`，并警告 ADB 端口 `5591` 超出旧版推荐范围 `[5555,5586]`，以及 build/boot properties 外部文件探测缺失。
- 配套 `adb devices -l` 结果为空（无 `device`、也无 `offline`），所以 `sys.boot_completed`、`init.svc.bootanim`、Android 版本均无法读取。

### Current conclusion

`BLACK_SCREEN_CLASS=D`（其他明确启动链错误，当前不能证明是 A 或 C）。最小下一步仅建议在用户确认实例结束后，用旧 Emulator 推荐 ADB 端口范围的单一对照启动，并确保只读观察 `adb=device`/`boot_completed`；本轮没有执行修复，不启动 APK，不改 ADB、网络、防火墙、APK、userdata、cache、VMware 或 C 盘，也不 wipe-data。

## Standard ADB port test (5554/5555)

- 为避免同一 userdata/cache 并行写入，先正常关闭了仍在运行的旧 5590/5591 实例；未强制终止。
- 仅建立独立启动器 `D:\CodexData\MagicWarrior\Start_MagicWarrior_HOST_QTFIX_5554.bat`，唯一变量是 `-ports 5554,5555`；仍使用同一 `runtime_qtfix`、API21 ARM32 system image、`gpu=host`、同一 userdata/cache；不启动 APK。
- 按顺序执行配套 `adb kill-server`、`adb start-server`，然后启动模拟器。5554/5555 启动前均空闲。
- 进程以 `Android Emulator - <build>:5554` 窗口标题运行并保持活动；观察超过 180 秒后正常关闭。
- `adb devices -l` 从初始空列表变为 `emulator-5554 offline`（非 absent，非 device）。因此标准端口消除了旧日志中的“5591 超出推荐范围”和启动时无法向 ADB server 注册的问题，但没有完成 ADB 握手。
- 新日志 `D:\CodexData\MagicWarriorCompat\matrix\host_qtfix_5554\emulator_console.log` 显示 `sent '0012host:emulator:5555' to ADB server`，随后停在 QEMU main loop/OpenGLES 初始化与 console 5554 监听；没有 `sys.boot_completed`、Android 版本、桌面或 EGL/GLES/FBO 错误。

### Current conclusion

`PORT_CHAIN_FIXED=PARTIAL`。5590/5591 的端口范围问题是真实阻塞，但改为 5554/5555 后仍是 `offline`，所以黑屏不能归因于端口问题本身，也不能证明是单纯 host GPU framebuffer；guest boot 仍未确认。按用户边界，本轮停止，不自动测试其他图形模式或修改任何游戏/系统文件。

## GPU-off control (5554/5555)

- 新建独立 `D:\CodexData\MagicWarrior\Start_MagicWarrior_OFF_QTFIX_5554.bat`；原 `HOST_QTFIX_5554.bat` 未修改。除 `-gpu off` 外，保持同一 Emulator 25.2.5、`runtime_qtfix`、API21 ARM32 system image、同一 userdata/cache、5554/5555 和其他参数。
- 按顺序执行配套 `adb kill-server`、`adb start-server`；启动前 5554/5555 无监听。
- 日志明确 `GPU emulation is disabled`、`qemu.gles=0`，并进入 QEMU main loop、监听 5554/5555。窗口进程保持运行超过 180 秒。
- `adb devices -l` 观察期间出现 `emulator-5554 offline`，180 秒结束仍为 `offline`，从未变为 `device`；因此未执行 getprop。
- 日志无 `sys.boot_completed`、Android 桌面或 EGL/GLES/FBO 错误；未启动 APK。观察结束后仅正常关闭本轮 emulator。

### Current conclusion

`HOST_GPU_BOOT_BLOCKER=NOT_CONFIRMED`。在完全相同的 runtime、镜像、userdata/cache 和标准端口下，`gpu=off` 仍长期 `offline`，所以 host GPU 不能单独解释 guest 未启动；当前更像共享的 API21 ARM32 guest boot/ADB offline 阻塞。

## Next candidate action

等待用户决定下一步；不要继续自动枚举 GPU。若继续，应先分析为什么 API21 ARM32 guest 在 QEMU 主循环后长期 ADB offline，再决定是否进行新的单变量测试。

## GitHub handoff sync

本轮 handoff 更新待提交并推送。

---

## QTFIX guest boot trace (read-only audit; no emulator launch)

`ROUND=QTFIX_GUEST_BOOT_TRACE`

本轮先按要求停止启动模拟器，比较了当前 `Start_MagicWarrior_OFF_QTFIX_5554.bat` / `Start_MagicWarrior_HOST_QTFIX_5554.bat` 与保存的 `start_api21_classic_offline.bat`、成功动态 logcat 及 D 盘基线副本。已确认成功基线存在，但当前 QTFIX off 不是同状态对照：核心 emulator/kernel/ramdisk/system/initdata 字节相同；userdata/cache、datadir、有效 hardware-qemu 配置、分区大小、DNS、窗口/快照参数、端口参考和 ADB 环境变量不同。

最关键差异：成功基线的 `matrix\gpu_off\userdata-qemu.img` SHA256=`6ab17f0e1f50418b66bf2380d6164057c1cd3f7a95d6af0c35a348527616eea2`、`cache.img`=`32f7385840bddb06fba014cc29ec88757edddc35a09d2edbd63374da37d07b30`；当前 QTFIX 脚本却硬编码使用 `matrix\gpu_host`，其 userdata=`cdec74029fe6d0523392ac86ef0ad4f57dd7f5bf249d6a87799c606d1f20dfec`、cache=`7a523466ae2b82717923f1c2cd27e5bcbfd3994d9e51342bb9fb493330d06f83`。大小相同但内容不同。当前有效 hardware-qemu 日志还显示 ncore=2、keyboard=false、audioInput/Output=true、camera.back=emulated，而成功 AVD 配置是 ncore=1、keyboard=yes、audioInput=no、audioOutput=no、camera.back=none。

文件核对：当前 QTFIX `emulator-arm.exe`=`3ca9ca373382b4998c81b72ef2ee3a2b8aa55b6dcffc7806fdbd32fe4d65ba36`（9,488,384 bytes）；kernel=`c0fb84b0ed4444a56abd3201bb1b13b3d8e664d50d604d7b82a54482d1be8bd6`（2,407,928）；ramdisk=`4e22f1e413580c3bd9f63a7564e2a24b0c783ff2f2fc5ba3ae3180589da7fe76`（713,673）；system=`f8eb24f36d2fac966b91fa497bfa1d6903241a8f53e3fbb0dd55063154c72bb7`（681,574,400）；initdata=`51f7ed47fcfe0a0f43bf3d36fcb70e4c33e3a5fd514199ef81bf418ac77a6a41`（576,716,800）。这些与 C 盘成功 API21 image / D 盘 baseline runtime 对应文件一致；没有发现 kernel/ramdisk/system 内容漂移。

现有 QTFIX off 日志只到 host 侧：`GPU emulation is disabled` → `Starting QEMU main loop` → 5554/5555 console/ADB 注册 → `Serial number ... emulator-5554`；无 kernel banner、init、mount、zygote/system_server、guest adbd 或 `sys.boot_completed`。成功基线 logcat 则有 `ActivityManager START`、`Start proc ... abi=armeabi`、`Displayed ... .battlelandAdr` 和 cocos2d-x 行，证明该保留组合曾进入 Android userspace。

本轮没有运行 `-show-kernel`，没有启动 APK，没有修改任何文件/网络/防火墙/VMware/C 盘，没有 wipe-data。详尽差异表与最终字段见 `handoff/CODEX_TO_CHAT.md` 的同名轮次。

### Round output

```makefile
ROUND=QTFIX_GUEST_BOOT_TRACE
KNOWN_GOOD_BOOT_CHAIN_FOUND=YES
CURRENT_VS_KNOWN_GOOD_DIFF_COUNT=12
SIGNIFICANT_BOOT_CHAIN_DIFF=当前 QTFIX 使用 gpu_host userdata/cache 与成功基线 gpu_off 不同，且有效 hardware-qemu、分区、DNS、窗口、快照和环境参数也不同；核心镜像字节相同。
CURRENT_EMULATOR_SHA256=3ca9ca373382b4998c81b72ef2ee3a2b8aa55b6dcffc7806fdbd32fe4d65ba36
CURRENT_KERNEL_SHA256=c0fb84b0ed4444a56abd3201bb1b13b3d8e664d50d604d7b82a54482d1be8bd6
CURRENT_RAMDISK_SHA256=4e22f1e413580c3bd9f63a7564e2a24b0c783ff2f2fc5ba3ae3180589da7fe76
CURRENT_SYSTEM_SHA256=f8eb24f36d2fac966b91fa497bfa1d6903241a8f53e3fbb0dd55063154c72bb7
CURRENT_USERDATA_SHA256=cdec74029fe6d0523392ac86ef0ad4f57dd7f5bf249d6a87799c606d1f20dfec
CURRENT_CACHE_SHA256=7a523466ae2b82717923f1c2cd27e5bcbfd3994d9e51342bb9fb493330d06f83
SHOW_KERNEL_TEST_RUN=NO
EMULATOR_WINDOW=NOT_RUN
ADB_DEVICE_VISIBLE=NOT_RUN
ADB_STATE=NOT_RUN
GUEST_KERNEL_STARTED=UNKNOWN
INIT_STARTED=UNKNOWN
SYSTEM_MOUNTED=UNKNOWN
DATA_MOUNTED=UNKNOWN
ANDROID_USERSPACE_STARTED=UNKNOWN
ZYGOE_OR_SYSTEM_SERVER_EVIDENCE=UNKNOWN
ADBD_GUEST_STARTED=UNKNOWN
SYS_BOOT_COMPLETED=NOT_AVAILABLE
KERNEL_PANIC=NO_EVIDENCE
FILESYSTEM_ERROR=UNKNOWN
INIT_FATAL_ERROR=UNKNOWN
LAST_KERNEL_BOOT_LINE=NOT_CAPTURED_THIS_ROUND
LAST_INIT_BOOT_LINE=NOT_CAPTURED_THIS_ROUND
LAST_ANDROID_USERSPACE_LINE=NOT_CAPTURED_THIS_ROUND
LAST_ADBD_LINE=emulator: sent '0012host:emulator:5555' to ADB server（host 注册，不代表 guest adbd）
ROOT_CAUSE_CLASS=G_BOOT_CHAIN_MISMATCH
ROOT_CAUSE_STATUS=CONFIRMED_SIGNIFICANT_DIFF_NOT_CAUSALLY_ISOLATED
MINIMAL_NEXT_TEST=在 D 盘新建隔离副本，采用 matrix\\gpu_off 已知成功 userdata/cache 与保存的 classic gpu=off 命令，仅增加 -show-kernel；不覆盖当前 QTFIX、成功基线或 C 盘文件。
```

PUSH_STATUS=SYNCED
