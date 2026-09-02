# 《魔界勇士》Windows 本地重建：PoC-0～3 验收报告

轮次：`WINDOWS_POC_0_TO_3`  
项目：`MagicWarrior_PC`（内部代号：土豆勇士）  
工程目录：`D:\CodexWorkspace\MagicWarrior_PC`

## 给非程序员的结论

第一阶段通过。已经生成一个实际可运行的 Windows `.exe`，它能打开窗口并用 Windows GDI+ 显示 APK 中的原始地图 PNG；原图没有出现 Android 模拟器里那种彩块/马赛克。程序还成功读取了原始 plist、JSON、地图 `.map` 和装备 `.sprite`，并把真实字段显示在窗口中。

这不是完整游戏，也没有进入战斗。当前 PoC 使用 WinForms/GDI+ 作为低成本 Windows 原生渲染验证；长期路线仍是现代 cocos2d-x/Axmol 兼容层。这样先证明资源和数据可用，再决定是否投入 PoC-4/5。

## 验收结果

| 项目 | 结果 | 证据 |
|---|---|---|
| PoC-0 Windows 程序 | PASS | `build\MagicWarrior_PC_PoC.exe`，SHA256 `a2f907e2725dbd2c0570c7c4aef3bb593b4483def8a295dd3f013799749a0a2c` |
| PoC-1 原始 PNG | PASS | `assets\UIImages\main_map_bg.png`，1024x512，SHA256 `ebefe7af8c48897f4521416ba72a0cd7c8d8bdd976ba4ce9fcd272c40a6dae22` |
| PoC-1 马赛克 | NO | GDI+ 解码和证据图 `evidence\poc1_render_evidence.png` |
| PoC-1 颜色/Alpha | YES / YES | 原 PNG 直接载入 `Bitmap`，未改像素 |
| PoC-2 plist | PASS | Apple plist XML；根节点 `plist`，55 个 key，抽样 frame `ditu_2` |
| PoC-2 JSON | PASS | UTF-8；顶层 49 个技能/面板条目，抽样 `skill33`、`skill19`、`TalentPanel` |
| PoC-3 map | PASS | `1-1.map` 为 XML-like `dict/key/string`，2,489 个 key，识别 `spriteName` 引用 |
| PoC-3 sprite | PASS | `iebgnauhz.sprite`；读取到真实装备“甲贺护手”及 `cnname/equipmenttype/level/property/suitId` |

## 技术路线

- PoC-0～3 实际实现：WinForms + System.Drawing/GDI+，通过 .NET Framework CSharp CodeDom 生成 Windows GUI EXE。
- 选择原因：当前机器没有现成 C++/Axmol 工具链；先用系统自带 .NET 完成可验证窗口、图片和数据读取，避免引入大依赖。
- 后续目标：迁移到 `ROUTE_B_MODERN_COCOS2DX_OR_AXMOL_WITH_COMPATIBILITY_DATA_LAYER`，保留验证过的资源/解析器行为。
- XML 解析明确关闭外部 DTD/网络解析（`XmlResolver=null`）；原 plist 的 Apple DTD 不会访问网络。

## 运行方式

双击：

`D:\CodexWorkspace\MagicWarrior_PC\Start_MagicWarrior_PC_PoC.bat`

程序读取工程 `assets` 目录中的复制品；原 APK 位于 `D:\CodexData\MagicWarriorCompat\game\magic_warrior.apk`，始终只读，未被修改。

## 本轮边界与最大问题

- 未启动 Android 模拟器、未连接服务器、未访问支付/账号/更新接口。
- 未修改 APK、未重签、未改 userdata/cache、未处理音频、未进入战斗。
- 原始 plist 含外部 Apple DTD；若使用默认 XML 解析器会尝试网络解析，本工程已用离线 XmlReader 设置消除该风险。
- `1-1.map` 的场景装配、碰撞和完整地图渲染仍属于 PoC-4；`.sprite` 字段已能稳定读取，但字段语义尚未全部反推。

## 下一步

`WINDOWS_NATIVE_RENDERING_PROOF=YES`，`LEGACY_DATA_READABILITY_PROOF=YES`。建议进入 PoC-4（静态地图场景装配）和 PoC-5（角色帧动画）之前，先由刀哥复核本报告和证据图；本轮硬停，不开始战斗重写。

## 机器可读状态

```ini
ROUND=WINDOWS_POC_0_TO_3
PROJECT_PATH=D:\CodexWorkspace\MagicWarrior_PC
FRAMEWORK=WinForms fallback (System.Drawing/GDI+); migration target Axmol/modern cocos2d-x
COMPILER=.NET Framework CSharp CodeDom provider
BUILD_SYSTEM=PowerShell Add-Type
POC0_STATUS=PASS
WINDOWS_EXE_GENERATED=YES
WINDOWS_EXE_PATH=D:\CodexWorkspace\MagicWarrior_PC\build\MagicWarrior_PC_PoC.exe
POC1_STATUS=PASS
ORIGINAL_ASSET_RENDERED=YES
MOSAIC_PRESENT=NO
COLOR_CORRECT=YES
ALPHA_CORRECT=YES
POC2_STATUS=PASS
PLIST_PARSE=PASS
JSON_PARSE=PASS
TEXT_ENCODING=UTF-8
SKILL_DATA_READABLE=YES
POC3_STATUS=PASS
MAP_PARSE=PASS
SPRITE_PARSE=PASS
EQUIPMENT_DATA_READABLE=YES
WINDOWS_NATIVE_RENDERING_PROOF=YES
LEGACY_DATA_READABILITY_PROOF=YES
ANDROID_EMULATOR_USED=NO
ORIGINAL_APK_MODIFIED=NO
NETWORK_SERVICE_USED=NO
ROUTE_B_FEASIBILITY_AFTER_POC=SUPPORTED
RECOMMEND_CONTINUE=YES (staged PoC only)
NEXT_STEP=PoC-4 static map scene assembly, then PoC-5 animation; no battle rewrite yet
```
