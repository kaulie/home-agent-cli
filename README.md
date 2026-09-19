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
