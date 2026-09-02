# 《魔界勇士》Windows 原生移植体检

审计轮次：`WINDOWS_NATIVE_PORT_FULL_AUDIT`  
范围：原始 APK 的只读结构、资源、DEX、ELF 动态符号/字符串，以及已经保存的离线运行证据。  
边界：没有修改 APK、没有重签/重打包、没有启动 APK、没有连接旧服务器，也没有修改模拟器、网络、userdata；C 盘未留下持久改动，临时分析辅助脚本已清理。

## 一句话结论

可以做，但这是 `CLASS_C_HEAVY_PORT`：图片、音乐、地图和大量数据可以保留；真正的战斗、角色、地图状态和存档规则主要在一个 ARM32 原生库里，必须重建 Windows 平台层并重新实现/验证这部分逻辑。它不是“直接编译 cocos2d-x 就完成”，但也不是只能从零重画。

## 审计源与完整性

权威分析副本：

`D:\CodexData\MagicWarriorCompat\game\magic_warrior.apk`

原始文件名为 `魔界勇士_2.1_APKPure.apk`；分析副本的 SHA256 与已保存的原 APK 一致：

`7977a38cdc22ae0c7f6c384feaba174961b7672c25fd08b1abf754c13e9e8cd6`

APK ZIP 统计：

| 项目 | 结果 |
|---|---:|
| 压缩文件大小 | 40,328,581 bytes |
| ZIP 条目总数 | 2,522 |
| 解压后总大小 | 63,705,791 bytes |
| `classes.dex` | 1 个，114,392 bytes |
| `lib/*.so` | 2 个 |
| `assets/` | 2,509 个 |
| `res/` | 5 个 |
| `META-INF/` | 3 个 |
| Windows PE（`.exe`/原生 Windows `.dll`） | 未发现 |

顶层目录/文件为 `AndroidManifest.xml`、`META-INF`、`assets`、`classes.dex`、`lib`、`res`、`resources.arsc`。

签名条目：`META-INF/MANIFEST.MF`、`META-INF/TEST.SF`、`META-INF/TEST.RSA`。这是 APK v1/JAR 签名；证书为 1024-bit RSA、`sha1WithRSAEncryption`，Subject/Issuer 为 `C=CN, ST=BJ, L=BJ, O=KONGZHONG, OU=KO, CN=SHENSY`，序列号 `0x4ff10401`，有效期 2012-07-02 至 2112-06-08。这里没有把未执行 `apksigner` 的 v2/v3 状态臆测为已确认。

## 游戏现在由什么组成

### APK 外壳和入口

`AndroidManifest.xml` 解析结果：

- package：`com.kongzhong.simlife.battlelandcn`
- versionName：`2.1`
- versionCode：`6`
- `minSdkVersion=8`
- 未声明 `targetSdkVersion`（`UNKNOWN`，不是假定为某个目标版本）
- 权限：`INTERNET`（重复出现两次）、`ACCESS_WIFI_STATE`、`READ_PHONE_STATE`、`WAKE_LOCK`、`WRITE_EXTERNAL_STORAGE`、`ACCESS_NETWORK_STATE`
- activity：`battlelandAdr`、`Main`、`InputBox`
- 主入口 intent-filter：`android.intent.action.MAIN` + `android.intent.category.LAUNCHER`，注册在 `battlelandAdr`
- receiver：`.BatteryAlarmReceiver`
- receiver intent-filter：`android.intent.action.ACTION_POWER_CONNECTED`、`android.intent.action.BATTERY_CHANGED`
- service：没有注册
- provider：没有注册

这说明 Android 部分是启动、窗口、输入、生命周期、设备信息和在线/支付包装层；它不是完整的玩法实现。电池 receiver 是实际 Manifest 注册项，但不等于开机自启动、远控或计划任务。

### Java/Dex

`classes.dex`：114,392 bytes，SHA256 `d0a95b2194491ca6cb17568beda35bb3df182ba1a5fbe518f4b75ff54aa56944`，97 个 class definitions、1,709 个字符串。

按类描述统计：

- `com.kongzhong.simlife.battlelandcn`：22 个（Activity、输入、IAP、receiver 和 R 类）
- `org.cocos2dx`：40 个（`Cocos2dxActivity`、`Cocos2dxRenderer`、`GLSurfaceView`、声音、位图、加速度计等）
- `com.noumena.android.dcpurchase`：20 个（支付 SDK）
- 其余为 Android 注解/壳类；Apache HttpClient 是使用到的 HTTP API，不作为独立游戏玩法类统计。

静态字符串/API 命中包括 `AssetManager`、`Activity`/`Context`、`GLSurfaceView`、EGL/GL10、`MediaPlayer`、`SoundPool`、`WebView`、`HttpGet`/`HttpPost`、`Socket`、`WifiManager`、`TelephonyManager`、`PowerManager` 和传感器。没有发现 `DexClassLoader` 或 `PathClassLoader` 字符串/API；动态 Dex 加载在本轮为 `NOT_CONFIRMED`，不能仅凭“有 classes.dex”声称存在。

Java 层结论：`MEDIUM_BUSINESS_LOGIC`，更准确地说是“薄 Android 外壳 + 在线/支付/输入胶水”，没有证据表明战斗规则主要在 Java。

### cocos2d-x

证据来自 `lib/armeabi/libgame.so` 字符串和符号，以及 DEX 中的 Cocos2dx Java 类：

- 明确字符串：`cocos2d-1.0.1-x-0.12.0`
- 符号/类：`CCDirector`、`CCEGLView`、`CCApplication`、`CCScene`、`CCSprite`、`CCTextureCache`、`CCAnimation`、`CCTMXLayer` 等
- Java glue：`Cocos2dxActivity`、`Cocos2dxRenderer`、`Cocos2dxBitmap`、`Cocos2dxSound`

因此 `COCOS2DX_PRESENT=YES`，版本置信度 `HIGH`，属于 C++ cocos2d-x + Android glue，不是 cocos2d-js 或 cocos2d-lua。  
`COCOS2DX_MODIFIED=UNKNOWN`：游戏集成和资源格式肯定是定制的，但两个 stripped 二进制无法证明上游 cocos2d-x 源码改了哪些行；不能把“定制集成”误写成“已证明修改了引擎源码”。

## 原生库审计

APK 只有 `lib/armeabi`，没有 `armeabi-v7a`、`arm64-v8a`、`x86` 或 `x86_64` 目录。两个库都是 ARM little-endian ELF32 `DYN`，没有 `.symtab`，只保留动态符号，不能直接当作可移植 Windows C++ 源码。

| 文件 | 大小 | SHA256 | 动态符号 / 定义导出 | NEEDED | 作用判断 |
|---|---:|---|---:|---|---|
| `lib/armeabi/libgame.so` | 7,481,752 | `5e7c03ea3e3c2c198c69d5bf96fba28c23947f4f6744abace3b92888d4deb412` | 10,745 / 10,462；JNI 24 个 | `libcocosdenshion.so`, `liblog.so`, `libz.so`, `libGLESv1_CM.so`, `libstdc++.so`, `libm.so`, `libc.so`, `libdl.so` | 核心游戏、渲染调用、资源读取、存档、联网和 Android JNI |
| `lib/armeabi/libcocosdenshion.so` | 87,280 | `112f552bef95afca1636742be92930748f9f2e232656a6f8833abc3d761b48d5` | 403 / 375；JNI_OnLoad | `liblog.so`, `libstdc++.so`, `libm.so`, `libc.so`, `libdl.so` | CocosDenshion 音频引擎 |

`libgame.so` 的 JNI 导出包括 `nativeInit`、`nativeRender`、`nativeOnPause`/`nativeOnResume`、触摸/按键、`nativeSetPaths`、`nativeSetDeviceInfo`、渠道和时间偏移、输入框，以及支付回调。这是明确的 Android/Java/GL 生命周期边界。

核心类/模块字符串包括：

- 数据/角色：`HBDataBase`、`HBRole`、`HBHero`、`HBRoleProperty`
- 战斗：`HBBattleRole`、`HBBattleLayer`、`BattleCommand`、`BattleQueryCommand`、`BattleResultCommand`
- 地图/UI：`HBMap`、`HBMapLayer`、`HBMapLayerData`、`HBMapEvent`、`MainMenu`、`CreateRoleLayer`、`MapScene`、`MapResultLayer`、`RoleInfoLayer`、`RoleSkillLayer`
- 技能：`HBSkill`、`HBSkillInforPanel`，以及 `animationScript/skill`、`skill/001` 至 `skill/041` 等资源
- 网络：`HBNet`、`CurlNetManager`、`GetServerCommand`、`GetRecordCommand`、`AddBattleRecordCommand`、`RankBattleCommand`、`RivalsUpdateCommand`
- 资源/引擎：`AppDelegate`、`CCDirector`、`CCTextureCache`、`CCAnimation` 等

另外，库中包含 `CLIENT libcurl 7.21.4` 和大量 libcurl HTTP 错误字符串。结论是 `NATIVE_LOGIC_CONCENTRATION=VERY_HIGH`：真正的玩法核心不能直接从 Android APK 搬成 Windows 可执行文件；需要对行为做逆向取证后重新实现，或尝试极不稳定的 ARM 翻译路线。

## 脚本与数据外置程度

完整扩展名扫描结果（`assets/`）：

| 类型 | 数量 | 说明 |
|---|---:|---|
| `.png` | 1,365 | UI、地图、角色、特效、图标和 atlas 图片 |
| `.jpg` | 27 | 背景/场景图片 |
| `.plist` | 545 | texture atlas、动画、UI 文案和粒子描述 |
| `.map` | 121 | 地图/UI 的 XML-like 自定义格式 |
| `.act` | 126 | 动作/特效描述 |
| `.ani` | 29 | 动画描述 |
| `.ant` | 5 | 动画/特效描述 |
| `.par` | 22 | 粒子参数 |
| `.sprite` | 131 | sprite/表格类描述，部分包含 JSON-like 数据 |
| `.json` | 15 | 技能、塔、NPC、竞技场等数据 |
| `.mp3` | 116 | 音乐和战斗音效 |
| `.wav` | 5 | UI 点击/确认类音效 |
| `.txt` | 1 | `assets/UI/uiScript/introJson.txt` |

没有发现 `.lua`、`.luac`、`.js`、`.jsc`、`.csv`、`.db`、`.sqlite`、`.proto` 或 `.bytes`。因此：

- `LUA_PRESENT=NO`
- `JAVASCRIPT_PRESENT=NO`
- 可执行脚本逻辑为 `NONE`；高数量的 plist/map/json/act/ani/par 是数据描述，不是脚本语言。机器字段使用 `SCRIPT_LOGIC_VOLUME=LOW`，并在此处明确“数据描述很多，规则仍在 native”。
- `DATA_EXTERNALIZATION_STATUS=HIGH`：图片、地图、动画、UI、技能描述、装备/物品字段和大量文案已外置；属性公式、战斗流程、AI、存档编码等仍明显依赖 native。

代表性资源证据：

- `assets/ChessBoardRes/map/1-1.map`：189,824 bytes，SHA256 `a1f342eb2b4e43bef2fe4a6f0aae272a33335f566383ce626092d292cdba74fb`，开头为 XML 声明。
- `assets/UIImages/main_map_bg.png`：317,485 bytes，SHA256 `ebefe7af8c48897f4521416ba72a0cd7c8d8bdd976ba4ce9fcd272c40a6dae22`。
- `assets/UIImages/main_map_bg.plist`：4,445 bytes，SHA256 `907a34c956f7f6dc9f191525fd0292c271848f4b3a4fbca290fa5c918734a730`。
- `assets/animationScript/battleRole0.plist`、`battleRole1.plist`：战斗角色帧/动作描述。
- `assets/word/SkillDescribe.json`：93,285 bytes，SHA256 `6df6d541b66d88165b70110abf5db82c8ba3bafbbac42ca0d9ff752248b91c8f`。
- `assets/ChessBoardRes/sprite/iebgnauhz.sprite`：可见装备字段如 `cnname`、`equipmenttype`、`level`、`property`、`suitId`，说明至少一部分装备表在客户端。

有一个名字带尾随空格的条目 `assets/UI/baoshishengji-hd.map `；这是 ZIP 打包命名瑕疵，不是缺失或恶意文件。

地图不是标准 `.tmx` 目录；它是自定义 XML-like `.map`，并与 `.plist`/`.sprite`/图片互相引用。地图数据可解析和复用，但场景装配、坐标、碰撞、UI 和渲染关系需要新客户端适配，因此 `MAP_DATA_REUSABLE=PARTIAL`，场景数据同理。

## 图片、动画与马赛克判断

静态资源没有发现 PVR、ETC、PKM、DDS、KTX 或 WebP 主纹理；PNG/JPEG/atlas 头部抽样合法，ZIP 解压正常。资源抽样不是逐像素验证，所以“所有图片都完美”仍不应声称为绝对事实，但当前证据足以把资源损坏排到低优先级。

`libgame.so` 同时使用：

- GLES 1.x 固定管线：`glMatrixMode`、`glOrthof`、`glFrustumf`、`glTexImage2D`、`glDrawElements`
- OES/FBO：`glGenFramebuffersOES`、`glBindFramebufferOES`、`glFramebufferTexture2DOES`、`glCheckFramebufferStatusOES`
- `glGenerateMipmapOES`、`glCompressedTexImage2D`、`glReadPixels`、VBO/texture atlas

历史动态证据显示：同一 APK 在 Android 5.0.2/API21 ARM32 曾启动、建角、进地图和启动首场战斗；地图大致能画出，战斗出现白屏和彩块，并伴随旧模拟器的 EGL/OES/FBO 兼容问题。这个组合更符合旧 Goldfish/模拟器 GLES、FBO、atlas 或 blend 路径问题，不符合“PNG 全部损坏”。

结论：

- `GRAPHICS_ASSET_INTEGRITY=HIGH`（ZIP/头部/抽样层面的高可信，不等于逐像素全量审计）
- `TEXTURE_FORMAT_RISK=MEDIUM`；没有常见专用压缩纹理，但旧 OES/FBO/atlas 路径有风险
- `SHADER_PORT_RISK=MEDIUM`；代码以固定管线为主，但仍有混合、粒子和 FBO 语义
- `RENDERING_PORT_RISK=HIGH`
- `MOSAIC_CAUSED_BY_ASSET_CORRUPTION=UNLIKELY`
- `WINDOWS_NATIVE_RENDERER_CAN_BYPASS_CURRENT_MOSAIC=LIKELY`

Windows/SDL/Axmol/现代 cocos2d-x 走正常 PNG/JPEG 解码和桌面 GL/D3D 后，理论上可以绕过当前“模拟器翻译层”的彩块；但必须重新实现纹理坐标、alpha、blend、FBO、粒子和 atlas 行为，不能只复制图片就保证画面一致。

## 音频

资源是 116 个 MP3 + 5 个 WAV；没有 OGG/M4A/CAF/MIDI。`libcocosdenshion.so` 导出 `SimpleAudioEngine`，Java 层同时有 `Cocos2dxMusic`、`Cocos2dxSound`、Android `MediaPlayer`/`SoundPool`。

文件本身可复用：`AUDIO_ASSETS_REUSABLE=YES`。但旧 Cocos2d-x 通过 `AssetManager.openFd()` 读取压缩 WAV 会失败；历史 logcat 已出现 “probably compressed”、MP3 resync 和 MediaPlayer `-38`。Windows 版需要自己的解码/播放后端，音频端口风险 `MEDIUM`；本轮不改音频资源。

## 存档系统

历史离线运行前后对 `/data/data/com.kongzhong.simlife.battlelandcn/` 和外部存储的只读比较显示：

- 没有 `shared_prefs/`、`files/`、`databases/` 中的可见存档；`cache/` 为空。
- 包根目录存在 `a_file_android_1_0`（1,680 bytes，二进制/疑似加密，人物与进度候选）。
- 包根目录存在 `a_server_android`（896 bytes，二进制/疑似加密，服务器状态候选）。
- `name_android_tmp`（112 bytes XML）内容含 `a_file_android_1_0=Player`，确认创建角色名写到本地。
- `xp.txt`（38 bytes XML）也落盘。
- 未发现 SQLite 数据库，也没有新的外部存档目录。

所以：

- `SAVE_SYSTEM_TYPE=package-private native binary + small XML metadata; no SQLite`
- `SAVE_LOCAL=YES`
- `SAVE_DATA_REUSABLE=PARTIAL`
- `SAVE_PORT_RISK=HIGH`

Windows 端可以保留原文件作为导入/备份格式；要做到无损继续游戏，需要找到 native 编解码规则，或设计一次性迁移器并与原 Android 结果逐字段比对。不能因为文件名可见就假定二进制已经解密。

## 网络与死服务器影响

静态字符串/符号确认了三类网络：

1. 游戏/记录类：`android.simlife.net/HeroBattle.php`、`android2.simlife.net/HeroBattle.php`、`www.simlife.com/HeroBattle_test/HeroBattle.php`。
2. 更新/渠道类：`http://herobattlecrack.simlife.net/updateherobattle.php?channel=`、`http://wmch.kongzhong.com`。
3. 支付类：`passport.kongzhong.com/m/pay.do?m=toPay`、`pay.noumenainnovations.com/pay/*`、`www.simlife.com/HeroBattle_pay/creakpay.php`、`www.simlife.com/admin/iap/*`。

历史完全断网动态证据表明：首屏、创建角色、地图、竞技场本地对手列表和首场战斗启动均能到达；只观察到 DNS/更新线程失败，没有连接旧官方服务器。长期在线竞技、排行榜同步、支付、更新、服务器结算没有做成功性保证。

结论：

- `NETWORK_REQUIRED_FOR_CORE_SINGLEPLAYER=NO`
- `DEAD_SERVER_BLOCKER_FOR_WINDOWS_PORT=PARTIAL`：死服务器会影响联网/支付/更新/排行等功能，但不是核心离线流程的启动阻塞。

Windows 离线版可以明确砍掉登录、支付、更新和在线排行，先做本地角色/地图/战斗；若以后要保留在线功能，再单独做受控 API，不把旧公网接口带进 PoC。

## Android/JNI 依赖地图

### 可以直接替换

- Android 路径字符串 -> Windows UTF-8/UTF-16 文件路径
- 基本时间、字符串、日志和线程封装
- 资源目录根路径
- 横屏窗口尺寸、输入坐标换算

### 需要兼容层

- `AssetManager`/`open()` -> ZIP 解包目录或只读资源包
- `SharedPreferences` 语义（虽然本 APK 未实际使用它存档）-> Windows 配置键值
- Cocos2d-x `CCFileUtils`、plist/json/XML 解析
- native 二进制存档读写和路径迁移
- 纹理 atlas、字体和粒子加载
- 音频资源查找/解码/播放

### 必须重写

- Windows entry point、窗口、消息循环、输入
- `Activity`/`Context`/`GLSurfaceView`/EGL 启动链
- 全部 JNI bridge（24 个 `libgame.so` JNI 导出）
- 设备信息、渠道、时间偏移、加速度计回调
- GLES1/OES/FBO renderer bootstrap 和帧缓冲语义
- Android MediaPlayer/SoundPool/CocosDenshion 平台后端
- `libgame.so` 中的角色、属性、战斗、AI、地图状态、技能、任务、掉落和存档规则

### 可以删除或隔离

- `BatteryAlarmReceiver`（离线桌面版不需要电池广播）
- `com.noumena.android.dcpurchase` 支付 SDK
- WebView/HTTP 支付页面
- 旧更新/渠道检查、账号/排行榜/在线记录模块
- 电话、Wi-Fi、设备 ID 和传感器等非核心能力

## 第三方 SDK 判断

静态可识别的独立商业 SDK 主要是 `com.noumena.android.dcpurchase`（20 个 DEX 类）。Cocos2d-x/CocosDenshion、libcurl 和 Android/Apache HTTP 是引擎或系统/底层库，不把每一个都夸大成商业 SDK。

估计：

- `THIRD_PARTY_SDK_COUNT=2`（Noumena DCPurchase + bundled libcurl；若只按商业 SDK 计则为 1）
- `OBSOLETE_ANDROID_SDK_COUNT=1`（支付 SDK）
- `REMOVABLE_SDK_COUNT=1`（支付/在线包装）
- `PORT_REQUIRED_SDK_COUNT=0`（离线核心 PoC）

这是“可识别数量”，不是完整软件供应链清单；APK 没有调试符号或 Gradle 源项目，未知项保留为 `UNKNOWN`。

## Windows 路线比较

| 路线 | 实现难度 | 内容复用 | 长期稳定性 | 画面兼容 | 逆向/重写 | 维护成本 | 结论 |
|---|---|---|---|---|---|---|---|
| A. 恢复/重建原 cocos2d-x Windows target | 高 | 资源高，代码低 | 中 | 中-高 | 很高（没有工程/源码） | 高 | 可作为参考，但不适合作为第一步 |
| B. 现代 cocos2d-x/Axmol + 数据兼容层 | 中高 | 资源/数据高，native 逻辑需移植 | 高 | 高 | 高但可分阶段 | 中 | **推荐** |
| C. ARM32 translation/compatibility bridge | 很高 | 理论上代码高 | 低 | 低-不确定 | 桥接本身很高 | 很高 | 不推荐，仍绕不开 JNI/GL/音频 |
| D. 全新 Windows client，APK 仅作规格/资源 | 很高 | 资源/规格高 | 高（完成后） | 高 | 最高，接近半重制 | 中高 | 无法恢复 native 行为时的后备路线 |

推荐唯一主路线：`ROUTE_B_MODERN_COCOS2DX_OR_AXMOL_WITH_COMPATIBILITY_DATA_LAYER`。原因是它能直接避开旧 Android emulator 的 GLES 黑屏/彩块，同时保留现有图片、地图、动画、文案和数据；native 玩法按模块逐步替换，而不是一次性假设所有行为。

## 可复用率估计

这些是工程估计区间，不是精确统计，也不把“二进制能运行”冒充“源码可复用”：

| 部分 | 估计 | 依据 |
|---|---:|---|
| ART_REUSE_PERCENT | 85-95% | PNG/JPG、atlas、UI art、角色头像和特效文件完整 |
| AUDIO_REUSE_PERCENT | 90-100% | MP3/WAV 文件可留，播放后端要换 |
| MAP_REUSE_PERCENT | 70-85% | 121 个自定义 map 可留，解析/场景装配要写 |
| DATA_REUSE_PERCENT | 70-85% | 技能、装备、文案和 UI 数据外置；公式/规则部分在 native |
| SCRIPT_REUSE_PERCENT | 50-70% | 没有可执行 Lua/JS；plist/map/json 描述可复用但需新解析器 |
| NATIVE_LOGIC_REUSE_PERCENT | 10-25% | ARM32 stripped `.so` 不能直接链接到 Windows，只能复用行为证据/符号/部分算法思路 |
| OVERALL_CONTENT_REUSE_PERCENT | 70-85% | 内容资源和数据为主；不代表客户端代码复用率 |

## 必须重写、可以移植、直接复用

### 必须重写

Windows 入口/窗口循环、输入、文件系统和存档路径、Android lifecycle/JNI、GLES/EGL/FBO renderer、音频后端、资源寻址，以及 `libgame.so` 的角色/属性/战斗/AI/地图状态/任务/技能/存档核心。

### 可以移植

plist/map/json/XML/sprite 解析器、资源索引、atlas 坐标、UI 结构、碰撞/地图数据模型、技能/动画时序、Cocos 场景树的概念和可从符号/运行日志恢复的行为。

### 可以直接复用

原始 PNG/JPG/MP3/WAV、atlas/动画描述、地图和文案数据、字体/图标，以及不涉及 Android 路径的内容命名。

### 可以删除

支付、在线账号、更新、旧排行榜/记录同步、设备 ID、电话/Wi-Fi/传感器和电池广播；是否保留要等离线版功能清单确定。

## Windows PoC 最小路线

本轮不开始大规模重写，只定义可验收的阶梯：

| 阶段 | 输入 | 需要实现 | 成功判据 | 最大风险 |
|---|---|---|---|---|
| PoC-0 | 无 | Win32/SDL/Axmol 空窗口和输入 | 能打开、关闭、横屏比例稳定 | 框架/编译工具链选择 |
| PoC-1 | `assets/MainMenu/server_icon.png` 或 `assets/UIImages/main_map_bg.png` | PNG/JPG 解码、alpha、纹理上传 | 图片无彩块、无拉伸错位 | atlas/alpha 语义 |
| PoC-2 | `assets/UIImages/main_map_bg.plist`、`assets/word/SkillDescribe.json` | plist/json/字体/UI 文案 | 显示一页菜单/文字 | 自定义字段与编码 |
| PoC-3 | `assets/ChessBoardRes/map/1-1.map`、一个 `.sprite` | map/sprite/数据索引解析 | 输出可验证的地图节点/装备字段 | 自定义 schema 和引用关系 |
| PoC-4 | `assets/UIImages/main_map_bg.plist` + map | 场景装配、相机和层级 | 显示一张原地图静态场景 | 坐标、裁剪、碰撞 |
| PoC-5 | `assets/animationScript/battleRole0.plist`、`battleRole1.plist`、一个 `.act/.ani` | 角色帧动画与粒子 | 角色待机/攻击动画完整 | 时序、blend、FBO |
| PoC-6 | 主菜单资源 | 离线菜单和角色选择状态 | 进入本地角色/地图入口 | 原生状态机缺源码 |
| PoC-7 | 地图节点 + 本地角色数据 | 地图移动、触发和本地存档 adapter | 可走到第一战斗入口 | 存档格式/规则 |
| PoC-8 | 战斗动画 + 重建的战斗状态 | 首场战斗最小循环 | 普通攻击、技能、HP/UI、结束状态一致 | native 战斗/AI 是最大工作量 |

建议先做 PoC-0 至 PoC-3。只有图片、文字、地图解析都能稳定通过，才投入战斗规则；这能在较小成本内验证“保留内容、重建客户端”的路线是否值得。

## 是否值得继续

`WINDOWS_POC_FEASIBLE=YES`，`RECOMMEND_CONTINUE=YES`，但前提是按 PoC 阶梯推进并接受“核心 native 逻辑需要较多重建”。如果目标只是短期马上玩，现有 Android 保底环境仍是更低成本；如果目标是摆脱旧模拟器、解决图形兼容、长期离线保存，Windows 原生路线值得继续。

本轮最重要的判断：不是“资源坏了”，也不是“有 cocos2d-x 就能直接编译”。正确描述是“内容资产保存得很好，Android 外壳可替换，玩法核心在 ARM32 stripped native 中，属于重度但可分阶段的移植”。

## 补充机器字段

```ini
APK_FILE_COUNT=2522
APK_TOTAL_SIZE=40328581
DEX_COUNT=1
NATIVE_LIBRARY_COUNT=2
ASSET_FILE_COUNT=2509
RES_FILE_COUNT=5
SUPPORTED_ABI=armeabi
PRIMARY_ABI=armeabi
WINDOWS_NATIVE_BINARY_PRESENT=NO
COCOS2DX_VERSION_CONFIDENCE=HIGH
SCRIPT_ENCRYPTED=UNKNOWN (no executable script language found)
MAP_FORMAT=custom XML-like .map plus plist/sprite references
MAP_DATA_REUSABLE=PARTIAL
SCENE_FORMAT=custom map/plist/sprite/json descriptors
SCENE_DATA_REUSABLE=PARTIAL
TEXTURE_FORMAT_RISK=MEDIUM
SHADER_PORT_RISK=MEDIUM
SAVE_LOCAL=YES
SAVE_DATA_REUSABLE=PARTIAL
AUDIO_ENGINE=CocosDenshion SimpleAudioEngine plus Android MediaPlayer/SoundPool glue
AUDIO_PORT_RISK=MEDIUM
THIRD_PARTY_SDK_COUNT=2 (one commercial payment SDK plus bundled libcurl; commercial-only count is 1)
OBSOLETE_ANDROID_SDK_COUNT=1
REMOVABLE_SDK_COUNT=1
PORT_REQUIRED_SDK_COUNT=0 for offline core
TECHNICAL_FEASIBILITY=MEDIUM
```

## 机器状态块

```ini
ROUND=WINDOWS_NATIVE_PORT_FULL_AUDIT

APK_INTEGRITY=VERIFIED_SHA256_MATCH; APK_V1_JAR_SIGNATURE_PRESENT
COCOS2DX_PRESENT=YES
COCOS2DX_VERSION=cocos2d-1.0.1-x-0.12.0
COCOS2DX_MODIFIED=UNKNOWN

PRIMARY_ABI=armeabi (ARM32 ELF)
NATIVE_LIBRARY_COUNT=2
NATIVE_LOGIC_CONCENTRATION=VERY_HIGH

LUA_PRESENT=NO
JAVASCRIPT_PRESENT=NO
SCRIPT_LOGIC_VOLUME=LOW (no executable Lua/JS; many data descriptors)

JAVA_LAYER_ROLE=MEDIUM_BUSINESS_LOGIC (Android launcher/glue plus online/payment wrappers; gameplay core is native)

DATA_EXTERNALIZATION_STATUS=HIGH
MAP_DATA_REUSABLE=PARTIAL
GRAPHICS_ASSET_INTEGRITY=HIGH (ZIP/header/sample verified)
AUDIO_ASSETS_REUSABLE=YES

ANDROID_DEPENDENCY_LEVEL=HIGH
JNI_DEPENDENCY_LEVEL=HIGH

SAVE_SYSTEM_TYPE=package-private native binary files plus small XML metadata; no SQLite
SAVE_PORT_RISK=HIGH

NETWORK_REQUIRED_FOR_CORE_SINGLEPLAYER=NO
DEAD_SERVER_BLOCKER_FOR_WINDOWS_PORT=PARTIAL

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
RECOMMEND_CONTINUE=YES

ROOT_CONCLUSION=Possible without Android Emulator, but it is a class-C heavy port: content is reusable, while the stripped ARM32 native gameplay core and Android rendering/lifecycle boundary must be rebuilt.
NEXT_STEP=Implement only PoC-0 through PoC-3 in a separate Windows prototype: window, one original image, one UI descriptor, and one map parser; do not start full battle rewrite yet.
```

## 证据限制

本轮没有完整反汇编 `libgame.so`，没有声称恢复了所有公式/AI/存档字段；没有把字符串命中当作运行行为；没有把未声明的 Manifest 组件、动态 Dex、Windows 现成工程或完整的在线结算能力臆测为存在。所有 `UNKNOWN` 和百分比区间都保留了这一限制。为生成本轮报告曾在 Codex 工作区短暂创建分析辅助脚本，现已删除，未留下 C 盘持久改动。
