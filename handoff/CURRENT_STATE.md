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

Codex sandbox 没有交互式 Windows GUI。此前用 D 盘副本前台启动时，emulator-arm.exe 进程约 30 秒仍存活，没有自然退出；因此无法在 sandbox 判断用户管理员桌面是否会显示窗口。旧 BAT 的 `start` 分离启动还会隐藏真实 stdout/stderr/exit code。

## Next candidate action

用户在正常 Windows 桌面双击前台启动器，等待最多 60 秒，只观察窗口和 Android 桌面，不启动游戏。窗口出现后回复 `HOST_WINDOW=YES`；没有窗口则回复 `HOST_WINDOW=NO`。下一轮再决定是否检查 ADB 和应用级网络隔离。

## GitHub handoff sync

本地 handoff 提交已完成；用户已完成浏览器认证，但随后本机 Git push 与只读 `ls-remote` 均无法连接 `github.com:443`，GitHub 写入接口仍返回 `HTTP 403 Resource not accessible by integration`。网络或连接器恢复后再执行 `git push origin main`。
