# home-agent-cli

home-agent 的**客户端 App**（iOS / Android）。

> 这些是**装在手机上的应用（App）**，不是常驻服务 —— 因此**不进** `agent-control-plane-deployment` 的部署系统
> （平台只打包 / 重启 `~/runtime/*` 下的服务；App 由 Xcode / Android Studio / adb 装到真机）。
> 本仓由 [home-agent-os](https://github.com/kaulie/home-agent-os) 迁出，**历史与目录结构都保留**（见文末「迁移说明」）。

## 目录

### iOS（`ios/`）

| 工程 | 是什么 |
|---|---|
| `LivingRoomEdge/` | **主 Console**（iOS 16+，SwiftUI）：Intent Source + Endpoint + 本机 Runtime；对话 / 扫描 / 拍照 / 文件 / 录音 / 直播 六个页 |
| `LivingRoomLegacy/` | iOS 12（UIKit）轻量版：老机器用；文字发 Intent、看书拍照、直播推流（P1 仅视频） |
| `LivingRoomPickup/` | iOS 12 拾音 App：麦克风 PCM 经 HAP1 推到家里 Mac `voice.stream`（默认 `:8792`） |
| `HomeAgentAdmin/` | Business Console：看谁在线、开关 Role / Runtime 能力、看 Runtime 的 intent 事件流 |
| `HomeAgentDev/` | Dev Console：看用户报的 Issue、跟踪 Dev Task / Agent 分析、把开发任务下发到 Mac Cursor Agent |
| `HomeAgentPickup/` | 拾音终端：mDNS 自动发现网关 + 音频 TCP 上行 + 反馈附件 |
| `HomeAgentRelay/` | 最小 HTTP / WebSocket 中继（验证用） |

每个工程自带 README（`ios/<App>/README.md`），协议细节以那里为准；工程文件可用各自的 `generate_xcodeproj.py` 重生成。

### Android（`android/`）

一个 Gradle 工程（`settings.gradle.kts` 里 `rootProject.name = "LivingRoomControl"`），含三个模块：

| 模块 | applicationId | 是什么 |
|---|---|---|
| `:living-room-android` | `com.smarthome.livingroom_android` | HomeAgent Console（Android）：与 iPhone Console 对齐的 Intent Source + Endpoint + 本机 Runtime |
| `:app-v2` | `com.smarthome.livingroom_v2` | Living Room Control v2（Edge Agent + Skill） |
| `:app` | `com.smarthome.livingroom` | v1（历史版本，保留参考） |

## 构建

```bash
# iOS —— 各工程自带 .xcodeproj
open ios/LivingRoomEdge/LivingRoomEdge.xcodeproj

# Android —— 用仓库里的 wrapper
cd android && ./gradlew :living-room-android:assembleDebug
```

> ⚠️ `:app-v2` 的源码跨到了**主仓** `plugins/`（`plugins/netease-music/android`、
> `plugins/chromecast-display/android`，即 `NetEaseMusicSkill` / `ChromecastDisplaySkill`）——
> 本仓是 `ios/` + `android/` 的快照，**没有这个目录**。本地编 `:app-v2` 时要先把它挂进来
> （CI 也是这么做的）：
>
> ```bash
> git clone --depth 1 https://github.com/kaulie/home-agent-os /tmp/home-agent-os
> ln -s /tmp/home-agent-os/plugins plugins    # 与 android/ 同级
> ```
>
> `:app`、`:living-room-android` 不依赖 `plugins/`，可直接编。

## CI（构建冒烟）

`.github/workflows/ci.yml` —— **只验证「能不能编过」**：不签名、不装真机、不部署（本仓没有部署平台的服务）。

| job | 做什么 |
|---|---|
| `ios` | 7 个工程**并行矩阵**（`fail-fast: false`，一次看全）：`xcodebuild -list` + 模拟器 SDK 的 Debug 编译（`CODE_SIGNING_ALLOWED=NO`） |
| `android` | `:app:assembleDebug` + `:app-v2:assembleDebug` + `:living-room-android:assembleDebug`、`:app-v2:testDebugUnitTest`；先 `clone` 主仓并 symlink `plugins/`（见上） |

日志里会先打印 Xcode / SDK / Java / Gradle 版本；Android 的 debug APK 作为 artifact 上传（便于手装真机）。

本地等价命令：

```bash
# iOS（逐个工程）
xcodebuild -project ios/LivingRoomEdge/LivingRoomEdge.xcodeproj -scheme LivingRoomEdge \
  -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' \
  CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY="" build

# Android（先按上面的方式挂好 plugins/）
cd android && ./gradlew :app:assembleDebug :app-v2:assembleDebug :living-room-android:assembleDebug
```

## 与其它仓库的关系

- **协议 / 契约**（HTTP 路由、intent / asset schema、能力声明）在 `home-agent-os`：
  `server/`（Brain）、`mac/`（Mac Edge）、`docs/project-map/`。客户端只实现
  「发出 intent → 轮询 / 接收结果 → 本机 runtime 执行」这一侧；**改协议要两侧一起改**。
- 本仓**不参与部署**：没有 `build.sh` / `scripts/restart.sh`，部署平台里也没有对应服务。

## 迁移说明

- 来源：`home-agent-os` 的 `main`（迁移时 `e02b42d`）。
- 方式：`git filter-branch --prune-empty` 只保留 `ios/`、`android/` 两个路径的历史
  （其它目录的提交被 prune 掉，保留下 **77 个提交**），**目录结构与原仓完全一致**，
  `git log --follow <文件>` 可继续追溯；`home-agent-os` 侧同步删除这两个目录并留指向本仓的说明。
