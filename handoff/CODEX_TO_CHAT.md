# Codex -> ChatGPT handoff

ROUND=WINDOWS_NATIVE_PORT_FULL_AUDIT

REPORT=handoff/WINDOWS_NATIVE_PORT_AUDIT.md
MACHINE_REPORT=handoff/WINDOWS_NATIVE_PORT_AUDIT.json

APK_INTEGRITY=VERIFIED_SHA256_MATCH; SHA256=7977a38cdc22ae0c7f6c384feaba174961b7672c25fd08b1abf754c13e9e8cd6
COCOS2DX_PRESENT=YES
COCOS2DX_VERSION=cocos2d-1.0.1-x-0.12.0 (HIGH confidence)
COCOS2DX_MODIFIED=UNKNOWN (custom game integration proven; upstream source delta not provable from stripped ELF)
PRIMARY_ABI=armeabi / ARM32
NATIVE_LIBRARY_COUNT=2
NATIVE_LOGIC_CONCENTRATION=VERY_HIGH (libgame.so 7,481,752 bytes; 10,462 defined dynamic exports; gameplay symbols/classes present)
LUA_PRESENT=NO
JAVASCRIPT_PRESENT=NO
SCRIPT_LOGIC_VOLUME=LOW (no executable Lua/JS; many plist/map/json/act/ani data descriptors)
JAVA_LAYER_ROLE=MEDIUM_BUSINESS_LOGIC (launcher/lifecycle/input/online/payment glue; gameplay core native)
DATA_EXTERNALIZATION_STATUS=HIGH
MAP_DATA_REUSABLE=PARTIAL
GRAPHICS_ASSET_INTEGRITY=HIGH (ZIP/header/sample checks; no PVR/ETC/PKM/DDS/KTX/WebP)
AUDIO_ASSETS_REUSABLE=YES
ANDROID_DEPENDENCY_LEVEL=HIGH
JNI_DEPENDENCY_LEVEL=HIGH
SAVE_SYSTEM_TYPE=package-private native binary files plus small XML metadata; no SQLite
SAVE_PORT_RISK=HIGH
NETWORK_REQUIRED_FOR_CORE_SINGLEPLAYER=NO (historical fully offline boot/create/map/arena/first-battle-start evidence)
DEAD_SERVER_BLOCKER_FOR_WINDOWS_PORT=PARTIAL (online/payment/update/rank only)
RENDERING_PORT_RISK=HIGH
MOSAIC_CAUSED_BY_ASSET_CORRUPTION=UNLIKELY
WINDOWS_NATIVE_RENDERER_CAN_BYPASS_CURRENT_MOSAIC=LIKELY
ART_REUSE_PERCENT=85-95%
AUDIO_REUSE_PERCENT=90-100%
MAP_REUSE_PERCENT=70-85%
DATA_REUSE_PERCENT=70-85%
SCRIPT_REUSE_PERCENT=50-70%
NATIVE_LOGIC_REUSE_PERCENT=10-25%
OVERALL_CONTENT_REUSE_PERCENT=70-85%
PORT_CLASS=CLASS_C_HEAVY_PORT
WINDOWS_PORT_FEASIBILITY=MEDIUM
ENGINEERING_RISK=VERY_HIGH
RECOMMENDED_WINDOWS_NATIVE_PATH=ROUTE_B_MODERN_COCOS2DX_OR_AXMOL_WITH_COMPATIBILITY_DATA_LAYER
MUST_REWRITE_COMPONENTS=Windows entry/window/input; filesystem/save adapter; Android lifecycle/JNI; GLES/EGL/FBO renderer; audio backend; ARM32 native gameplay/state/AI/save logic; optional online/payment abstraction
WINDOWS_POC_FEASIBLE=YES
RECOMMEND_CONTINUE=YES (staged PoC only)
ROOT_CONCLUSION=Windows-native recovery is feasible without Android Emulator, but this is a class-C heavy port: content is reusable while stripped ARM32 gameplay and Android rendering/lifecycle boundaries must be rebuilt.
NEXT_STEP=PoC-0..PoC-3 only: open a Windows window, load one original PNG, render one UI descriptor, and parse one original map; do not start a full battle rewrite.

EVIDENCE_SUMMARY=
- APK copy: D:\CodexData\MagicWarriorCompat\game\magic_warrior.apk, 40,328,581 bytes, 2,522 ZIP entries, 2,509 assets, 5 res entries, 1 classes.dex, 2 ARM32 libraries, no Windows PE.
- Manifest: 3 activities (battlelandAdr MAIN/LAUNCHER, Main, InputBox), 1 BatteryAlarmReceiver (power-connected/battery-changed), no services/providers; minSdk 8, target SDK not declared.
- Native: libgame.so SHA256 5e7c03ea3e3c2c198c69d5bf96fba28c23947f4f6744abace3b92888d4deb412; libcocosdenshion.so SHA256 112f552bef95afca1636742be92930748f9f2e232656a6f8833abc3d761b48d5; both stripped ELF32 ARM.
- Content: 1,365 PNG, 27 JPG, 545 plist, 121 map, 126 act, 29 ani, 5 ant, 22 par, 131 sprite, 15 JSON, 116 MP3, 5 WAV; no Lua/JS.
- Historical dynamic evidence: API21/ARM32 offline install, boot, character creation, map entry and first-battle start succeeded; saved state is package-root binary/XML, not SQLite; server is not required for core single-player.
- Rendering evidence: fixed GLES1 + OES/FBO/mipmap/atlas calls and old emulator EGL/FBO errors; image headers are valid, so mosaic is unlikely to be asset corruption.

FILES_CREATED=handoff/WINDOWS_NATIVE_PORT_AUDIT.md; handoff/WINDOWS_NATIVE_PORT_AUDIT.json
FILES_UPDATED=handoff/CODEX_TO_CHAT.md; handoff/CURRENT_STATE.md
SCOPE=READ_ONLY_STATIC_ARCHITECTURE_AUDIT
APK_MODIFIED=NO
EMULATOR_STARTED=NO
NETWORK_ACCESSED=NO
C_DRIVE_WRITTEN=NO
LOCAL_COMMIT=8dc9eedc11ab45ea23ed35c5fbc69ac528fc9601
PUSH_STATUS=BLOCKED_NETWORK (github.com:443 unreachable from this session; no remote update confirmed)

---

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

ROUND=QTFIX_STANDARD_ADB_PORT_TEST
PORTS=5554,5555
ADB_SERVER_PRESTARTED=YES（按顺序执行配套 adb kill-server、adb start-server；初始 adb devices -l 为空）
EMULATOR_WINDOW=YES（emulator-arm.exe 以 `Android Emulator - <build>:5554` 窗口标题运行；测试结束后正常关闭）
ADB_DEVICE_VISIBLE=YES（约 20 秒后出现 `emulator-5554`；最终观察仍可见）
ADB_STATE=offline（不是 absent；`adb devices -l` 显示 `emulator-5554 offline transport_id`）
SYS_BOOT_COMPLETED=NOT_AVAILABLE（状态不是 device，未执行 getprop）
ANDROID_DESKTOP=NOT_CONFIRMED／NO_EVIDENCE（180 秒内无 ADB boot 属性，日志也没有 Android 桌面阶段；未启动 APK）
LAST_LOG_STAGE=已越过旧 5591 的 connection-refused 阶段并成功向 ADB server 发送 `host:emulator:5555`；随后仍停在 `Starting QEMU main loop`、`Initializing hardware OpenGLES emulation support`、监听 console 5554
GLES_EGL_ERRORS=NONE_REPORTED（无 EGL/GLES/FBO error、native crash 或 fatal 行；仅有既有的 build/boot properties、metrics/update-check 警告）
PORT_CHAIN_FIXED=PARTIAL（5591 超范围警告消失，5554/5555 注册链已可见；但 ADB 仍 offline，Android guest 未完成可观测启动）
ROOT_CAUSE_STATUS=PORT_RANGE_WAS_A_REAL_BLOCKER_BUT_NOT_THE_ONLY_BLOCKER（标准端口修复了 ADB server 注册/端口范围问题，却没有使 guest 进入 device/boot_completed；黑屏根因仍不能归为纯 GPU framebuffer）
NEXT_STEP=停止自动尝试；先报告。本轮不测 guest/software GPU、不换 Emulator、不改 APK/网络/userdata/cache/C 盘、不 wipe-data、不处理音频。
TEST_LAUNCHER_CREATED=D:\CodexData\MagicWarrior\Start_MagicWarrior_HOST_QTFIX_5554.bat（独立副本；原 QTFIX 启动器未覆盖）
TEST_LOG_PATH=D:\CodexData\MagicWarriorCompat\matrix\host_qtfix_5554\emulator_console.log
TEST_CLEANUP=YES（观察结束后仅正常关闭本轮 emulator；未强制杀进程）

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

---

ROUND=QT_WINDOWS_PLATFORM_PLUGIN
QWINDOWS_DLL_FOUND=NO
QWINDOWS_DLL_PATH=NOT_FOUND（目标路径 `D:\\CodexData\\MagicWarriorCompat\\runtime\\emulator\\lib\\qt\\plugins\\platforms\\qwindows.dll` 不存在；对旧 Emulator 目录的只读文件搜索也未找到可确认的 qwindows.dll）
QT_PLUGIN_PATH_BEFORE=UNSET（现有 BAT 只设置普通 PATH，未设置 QT_QPA_PLATFORM_PLUGIN_PATH）
QT_PLUGIN_PATH_TESTED=NOT_TESTED（DLL 缺失，未启动模拟器）
TEST_LAUNCHER_CREATED=NO
EMULATOR_WINDOW_RESULT=NOT_TESTED
EXIT_CODE=NOT_RUN
ROOT_CAUSE_STATUS=CONFIRMED_MISSING_LOCAL_QT_PLATFORM_PLUGIN
NEXT_STEP=不从随机网站下载或替换 DLL；先取得与 Emulator 25.2.5 匹配的官方包并确认其中是否有 `windows-x86/plugins/platforms/qwindows.dll`。官方 Android Qt 预编译目录可见该标准路径：https://android.googlesource.com/platform/prebuilts/android-emulator-build/qt/%2B/refs/heads/emu-2.0-release

---

ROUND=OFFICIAL_25_2_5_QT_RESTORE
OFFICIAL_PACKAGE_SOURCE=https://dl.google.com/android/repository/tools_r25.2.5-windows.zip
OFFICIAL_PACKAGE_DOWNLOADED=YES
OFFICIAL_PACKAGE_HASH=SHA256: da1a0bd9bb358cb52a8fc0a553a060428efe11151e69b9ea7a5cbacb27cf1c7c
QWINDOWS_IN_OFFICIAL_PACKAGE=YES（x86 tools\\lib；另有 x64 tools\\lib64 对应副本）
QWINDOWS_OFFICIAL_PATH=D:\\CodexData\\MagicWarriorCompat\\official_sdk_tools_25_2_5\\tools\\lib\\qt\\plugins\\platforms\\qwindows.dll
QT_RUNTIME_COMPLETE=YES（官方 x86 包含 12 个 Qt5 lib DLL 和 13 个 plugins 文件；platforms/imageformats/bearer/generic/iconengines/printsupport/sqldrivers 均存在）
CURRENT_RUNTIME_MISSING_FILES=整个 `runtime\\emulator\\lib\\qt\\plugins` 子树缺失（qwindows.dll、qminimal.dll、qgenericbearer.dll、qnativewifibearer.dll、qtuiotouchplugin.dll、qsvgicon.dll、qgif/qico/qjpeg/qsvg.dll、windowsprintersupport.dll、qsqlite/qsqlodbc.dll 等 13 个官方 x86 插件文件）
RUNTIME_QTFIX_CREATED=YES（D:\\CodexData\\MagicWarriorCompat\\runtime_qtfix；以旧 runtime 为基础，仅补齐官方 x86 plugins 子树）
QTFIX_LAUNCHER_CREATED=YES（D:\\CodexData\\MagicWarrior\\Start_MagicWarrior_HOST_QTFIX.bat）
ORIGINAL_RUNTIME_UNCHANGED=YES（原 `runtime\\emulator\\lib\\qt\\plugins` 仍不存在；未覆盖原文件）
NEXT_STEP=用户双击 QTFIX 启动器，观察是否出现 Android Emulator 窗口；本轮不启动 APK、不处理 ADB/网络/音频/guest。

---

ROUND=QTFIX_BLACK_SCREEN_LIVE_DIAGNOSIS
PROCESS_ALIVE=YES（PID 15644；tasklist 状态为 Running；窗口标题为 Android Emulator - <build>:5590；未观察到独立 qemu-system 进程，旧版 QEMU 嵌入 emulator-arm.exe）
CPU_ACTIVITY=YES（只读两次 tasklist，约 6 秒间隔，CPU 累计时间由 0:13:00 增至 0:13:06；进程未冻结）
ADB_STATE=NO_DEVICE（配套 adb devices -l 启动/连接了 adb server，但设备列表为空；没有 offline 条目）
SYS_BOOT_COMPLETED=NOT_AVAILABLE（无 ADB device，未执行 getprop）
LAST_LOG_STAGE=QEMU main loop 已启动；OpenGLES 初始化已开始；已监听 console 5590 与 ADB 5591，并报告 Serial number emulator-5590；日志随后无 Android boot 完成信息
GLES_EGL_ERRORS=NONE_REPORTED（日志只有“Initializing hardware OpenGLES emulation support”，未出现 EGL/GLES/FBO error、native crash 或 fatal 行）
GUEST_BOOT_STATUS=NOT_CONFIRMED（黑屏持续且 ADB 无设备，日志没有 sys.boot_completed/桌面阶段证据）
BLACK_SCREEN_CLASS=D（当前最佳分类：其他明确启动链错误；不能证明 A 或 C。日志明确记录启动时连接 ADB server 被拒绝、5591 超出旧版推荐范围 [5555,5586]，并且 QtFix runtime 的 build.prop/boot.prop 外部探测文件缺失；QEMU 主循环和 CPU 仍活动）
ROOT_CAUSE=尚不能把黑屏归咎于 GPU framebuffer：Android 是否完成 boot 未知。可确认的阻塞是旧 Emulator 的 ADB/控制端口链未形成设备（启动时 connection refused；5591 为非推荐端口），同时日志停在 QEMU/OpenGLES 初始化之后；没有直接 EGL/GLES 错误。
MINIMAL_NEXT_TEST=在用户桌面结束本次实例后，下一轮只建立不改 userdata 的可控启动对照：使用旧 Emulator 推荐 ADB 端口范围内的单一端口组合并先让配套 adb server 存在，再观察是否出现 ADB=device/boot_completed；若仍无设备，再单独测试 gpu host 与 guest/软件渲染的窗口启动差异。当前轮不执行修复。
ACTION_TAKEN=仅读取进程、CPU 累计时间、netstat、emulator_console.log 和 adb devices；未启动 APK、未执行 getprop、未改 ADB/网络/防火墙、未改 APK/userdata/cache/VMware/C 盘、未 wipe-data。
PUSH_STATUS=SYNCED

---

ROUND=QTFIX_GPU_OFF_CONTROL
PORTS=5554,5555
GPU_MODE=off（唯一变量；日志中的 `qemu.gles=0`，并明确 `GPU emulation is disabled`）
EMULATOR_WINDOW=YES（emulator-arm.exe 以 `Android Emulator - <build>:5554` 运行；测试结束后正常关闭）
ADB_DEVICE_VISIBLE=YES（观察期间出现 `emulator-5554`）
ADB_STATE=offline（180 秒观察结束仍为 offline，未出现 device）
SYS_BOOT_COMPLETED=NOT_AVAILABLE（没有 device 状态，未执行 getprop）
ANDROID_DESKTOP=NOT_CONFIRMED／NO_EVIDENCE（日志无 boot completed/桌面阶段；未启动 APK）
LAST_LOG_STAGE=GPU emulation disabled；QEMU main loop 已启动；console 5554 与 ADB 5555 已监听；日志没有继续进入 Android boot 阶段
HOST_GPU_BOOT_BLOCKER=NOT_CONFIRMED（gpu=off 在相同 runtime、镜像、userdata/cache、端口下也 offline，因此 host GPU 不能单独解释 guest 未启动）
ROOT_CAUSE_STATUS=GPU_NOT_SUFFICIENT；当前更像共享的 guest boot/ADB offline 阻塞，尚无 EGL/GLES 错误证据
NEXT_STEP=停止自动 GPU 枚举；先分析 API21 ARM32 guest 为什么在 QEMU 主循环后长期 ADB offline，再决定后续单变量测试
TEST_LAUNCHER_CREATED=D:\CodexData\MagicWarrior\Start_MagicWarrior_OFF_QTFIX_5554.bat（独立副本；原 HOST_QTFIX_5554 未修改）
TEST_LOG_PATH=D:\CodexData\MagicWarriorCompat\matrix\off_qtfix_5554\emulator_console.log
TEST_CLEANUP=YES（观察超过 180 秒后仅正常关闭 emulator；未强制终止）
PUSH_STATUS=SYNCED

---

ROUND=QTFIX_GUEST_BOOT_TRACE

SCOPE=READ_ONLY_STARTUP_CHAIN_AUDIT

KNOWN_GOOD_BOOT_CHAIN_FOUND=YES

KNOWN_GOOD_REFERENCE=
成功动态证据来自 `C:\Users\Administrator\Documents\Codex\2026-08-23\new-chat\start_api21_classic_offline.bat` 及其产生的 `dynamic_boot_logcat.txt`、人物/地图/战斗日志。该命令使用 C 盘官方 API21 ARM32 system image、`tools_old_25.2.5\tools\emulator-arm.exe`、独立 `android_avd\magic_api21` 数据目录和 `gpu=off`。成功数据镜像的 D 盘只读副本为 `D:\CodexData\MagicWarriorCompat\matrix\gpu_off` / `D:\CodexData\MagicWarrior\avd\magic_api21`。精确的那一次动态调用没有单独保存完整命令转储；因此端口以存档 BAT 的 5556,5557 为命令参考，动态报告另提到清理时曾有 5560,5561 临时实例。

KNOWN_GOOD_EMULATOR_VERSION=25.2.5-3567187（归档文件名/工具包与实际日志一致；`BASELINE.md` 中“30.2.2 / BuildId 6885378”与实际二进制/日志不一致，未把该段文字当作版本证据）

CURRENT_VS_KNOWN_GOOD_DIFF_COUNT=12

DIFF_COUNT_DEFINITION=仅计入启动器/有效硬件配置中已核实的差异；内容相同的核心镜像仍记录为路径差异，未把相同字节重复计为镜像内容差异。

DIFF_01=emulator/runtime 根目录不同：known-good 使用 `C:\Users\Administrator\Documents\Codex\2026-08-23\new-chat\android_sdk\tools_old_25.2.5\tools\emulator-arm.exe`；current 使用 `D:\CodexData\MagicWarriorCompat\runtime_qtfix\emulator\emulator-arm.exe`。二者 SHA256 相同。
DIFF_02=sysdir/system/kernel/ramdisk/initdata 根目录不同：known-good 为 C 盘 API21 system image；current 为 `D:\CodexData\MagicWarriorCompat\runtime_qtfix\system-image`。对应镜像字节相同。
DIFF_03=datadir/data/cache 根目录不同：known-good 为 `android_avd\magic_api21`（D 盘镜像副本 `matrix\gpu_off`）；current QTFIX 两个脚本都硬编码 `D:\CodexData\MagicWarriorCompat\matrix\gpu_host`。
DIFF_04=userdata-qemu.img 内容不同：known-good `6ab17f0e1f50418b66bf2380d6164057c1cd3f7a95d6af0c35a348527616eea2`；current `cdec74029fe6d0523392ac86ef0ad4f57dd7f5bf249d6a87799c606d1f20dfec`。
DIFF_05=cache.img 内容不同：known-good `32f7385840bddb06fba014cc29ec88757edddc35a09d2edbd63374da37d07b30`；current `7a523466ae2b82717923f1c2cd27e5bcbfd3994d9e51342bb9fb493330d06f83`。
DIFF_06=存档命令 `-partition-size 550`；current QTFIX 使用 `-partition-size 650`。
DIFF_07=存档命令 `-ports 5556,5557`；current QTFIX 使用 `-ports 5554,5555`（动态成功实例的实际端口未完整留档，故仅把保存命令作为可核对参考）。
DIFF_08=存档命令带 `-no-window`；current QTFIX 为可见 GUI，没有 `-no-window`。
DIFF_09=存档命令显式带 `-no-snapshot-save -no-snapshot-load`；current 只有 `-no-snapshot`。
DIFF_10=存档命令显式 `-dns-server 127.0.0.1`；current QTFIX 未传该参数，日志实际解析为 `192.168.5.1`。
DIFF_11=有效 hardware-qemu 配置不同：current QTFIX 日志显示 `hw.cpu.ncore=2`、keyboard=false、audioInput/Output=true、camera.back=emulated；known-good AVD config 为 ncore=1、keyboard=yes、audioInput=no、audioOutput=no、camera.back=none。classic QEMU 还明确提示 SMP 配置被忽略。
DIFF_12=环境变量集合不同：known-good 的 `android_env.bat` 设置了 `ANDROID_SDK_HOME`、`ANDROID_AVD_HOME`、`ADB_VENDOR_KEYS` 和 `ANDROID_ADB_SERVER_PORT=5038`；current QTFIX 设置 Qt 插件路径及部分 Android/HOME 变量，但没有 `ADB_VENDOR_KEYS`/`ANDROID_ADB_SERVER_PORT`，并额外带 `-verbose`。

SIGNIFICANT_BOOT_CHAIN_DIFF=
当前 QTFIX off 控制并不是已知成功状态的同状态对照：它使用了 `matrix\gpu_host` 的 userdata/cache，而成功基线是 `matrix\gpu_off` / `avd\magic_api21`；两套镜像大小相同但 SHA256 均不同。与此同时，QTFIX 使用 GUI、650MB 分区、未指定 DNS、5554/5555 端口和由前次 host 测试留下的有效 hardware-qemu 配置。核心 emulator、kernel、ramdisk、system、initdata 内容相同，因此目前最强证据是“数据/缓存及启动参数链不一致”，不是“API21 ARM32 镜像本身不可启动”。

## Core file audit (absolute paths, existence, size, SHA256)

| artifact | known-good path / evidence | current QTFIX path | result |
|---|---|---|---|
| emulator-arm.exe | `C:\Users\Administrator\Documents\Codex\2026-08-23\new-chat\android_sdk\tools_old_25.2.5\tools\emulator-arm.exe`, exists, 9,488,384 bytes, `3ca9ca373382b4998c81b72ef2ee3a2b8aa55b6dcffc7806fdbd32fe4d65ba36` | `D:\CodexData\MagicWarriorCompat\runtime_qtfix\emulator\emulator-arm.exe`, exists, 9,488,384 bytes, same SHA256 | bytes identical |
| kernel-qemu | `C:\Users\Administrator\Documents\Codex\2026-08-23\new-chat\android_sdk\system-images\android-21\default\armeabi-v7a\kernel-qemu`, exists, 2,407,928 bytes, `c0fb84b0ed4444a56abd3201bb1b13b3d8e664d50d604d7b82a54482d1be8bd6` | `D:\CodexData\MagicWarriorCompat\runtime_qtfix\system-image\kernel-qemu`, exists, 2,407,928 bytes, same SHA256 | bytes identical |
| ramdisk.img | C API21 image, exists, 713,673 bytes, `4e22f1e413580c3bd9f63a7564e2a24b0c783ff2f2fc5ba3ae3180589da7fe76` | `D:\CodexData\MagicWarriorCompat\runtime_qtfix\system-image\ramdisk.img`, exists, 713,673 bytes, same SHA256 | bytes identical |
| system.img | C API21 image, exists, 681,574,400 bytes, `f8eb24f36d2fac966b91fa497bfa1d6903241a8f53e3fbb0dd55063154c72bb7` | `D:\CodexData\MagicWarriorCompat\runtime_qtfix\system-image\system.img`, exists, 681,574,400 bytes, same SHA256 | bytes identical |
| initdata/userdata.img | C API21 image, exists, 576,716,800 bytes, `51f7ed47fcfe0a0f43bf3d36fcb70e4c33e3a5fd514199ef81bf418ac77a6a41` | `D:\CodexData\MagicWarriorCompat\runtime_qtfix\system-image\userdata.img`, exists, 576,716,800 bytes, same SHA256 | bytes identical |
| userdata-qemu.img | `D:\CodexData\MagicWarriorCompat\matrix\gpu_off\userdata-qemu.img` (same hash as C successful AVD and `D:\CodexData\MagicWarrior\avd\magic_api21`), exists, 681,574,400 bytes, `6ab17f0e1f50418b66bf2380d6164057c1cd3f7a95d6af0c35a348527616eea2` | `D:\CodexData\MagicWarriorCompat\matrix\gpu_host\userdata-qemu.img`, exists, 681,574,400 bytes, `cdec74029fe6d0523392ac86ef0ad4f57dd7f5bf249d6a87799c606d1f20dfec` | content differs |
| cache.img | `D:\CodexData\MagicWarriorCompat\matrix\gpu_off\cache.img` (same hash as C successful AVD and `D:\CodexData\MagicWarrior\avd\magic_api21`), exists, 69,206,016 bytes, `32f7385840bddb06fba014cc29ec88757edddc35a09d2edbd63374da37d07b30` | `D:\CodexData\MagicWarriorCompat\matrix\gpu_host\cache.img`, exists, 69,206,016 bytes, `7a523466ae2b82717923f1c2cd27e5bcbfd3994d9e51342bb9fb493330d06f83` | content differs |

## Existing current-log evidence (not a new launch)

Current QTFIX off log `D:\CodexData\MagicWarriorCompat\matrix\off_qtfix_5554\emulator_console.log` reaches only host-side QEMU setup:

- line 24: `emulator: GPU emulation is disabled`
- line 106: `emulator: Starting QEMU main loop`
- line 127: `emulator: (setup_console_and_adb_ports) trying console port 5554, adb port 5555 (legacy: true)`
- line 129: `emulator: sent '0012host:emulator:5555' to ADB server`
- line 130: `emulator: Listening for console connections on port: 5554`
- line 131: `emulator: Serial number of this emulator (for ADB): emulator-5554`
- no Linux kernel banner, init, mount, zygote, system_server, guest adbd, or `sys.boot_completed` line is present; this run did not use `-show-kernel`.

The known-good `D:\CodexData\MagicWarriorCompat\baseline\evidence\dynamic_boot_logcat.txt` proves a later Android userspace stage for the preserved baseline: `ActivityManager START ... .battlelandAdr`, `Start proc ... abi=armeabi`, `Displayed ... .battlelandAdr`, and `cocos2d-x debug info`. These are prior-run evidence, not evidence from the current QTFIX files.

## Required final state block

ROUND=QTFIX_GUEST_BOOT_TRACE

KNOWN_GOOD_BOOT_CHAIN_FOUND=YES
CURRENT_VS_KNOWN_GOOD_DIFF_COUNT=12
SIGNIFICANT_BOOT_CHAIN_DIFF=当前 QTFIX 使用的 gpu_host userdata/cache 与成功基线 gpu_off 不同，且有效 hardware-qemu、分区、DNS、窗口、快照和环境参数也不同；核心 emulator/kernel/ramdisk/system/initdata 字节相同。

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

KERNEL_PANIC=NO
FILESYSTEM_ERROR=UNKNOWN
INIT_FATAL_ERROR=UNKNOWN

LAST_KERNEL_BOOT_LINE=NOT_CAPTURED_THIS_ROUND（未运行 -show-kernel）
LAST_INIT_BOOT_LINE=NOT_CAPTURED_THIS_ROUND
LAST_ANDROID_USERSPACE_LINE=NOT_CAPTURED_THIS_ROUND；current log 无 userspace 行
LAST_ADBD_LINE=emulator: sent '0012host:emulator:5555' to ADB server（仅 host 注册，不是 guest adbd 已启动证据）

ROOT_CAUSE_CLASS=G_BOOT_CHAIN_MISMATCH
ROOT_CAUSE_STATUS=CONFIRMED_SIGNIFICANT_DIFF_NOT_CAUSALLY_ISOLATED
MINIMAL_NEXT_TEST=仅在下一轮获准后：在 D 盘建立新的只读基线副本，使用 `matrix\gpu_off` 的已知成功 userdata/cache 与保存的 classic gpu=off 命令，保持核心镜像不变，并只新增 `-show-kernel`；不覆盖当前 QTFIX、成功基线或任何 C 盘文件。

ACTION_TAKEN=只读审计 BAT、配置、日志、文件存在性/大小/SHA256；未启动模拟器，未创建或运行 SHOWKERNEL 启动器，未启动 APK，未修改 userdata/cache/APK/网络/防火墙/VMware/C 盘，未 wipe-data。

PUSH_STATUS=SYNCED
