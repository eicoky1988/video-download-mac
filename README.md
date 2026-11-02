# 视频下载 for macOS

一个把 [yt-dlp](https://github.com/yt-dlp/yt-dlp) + [ffmpeg](https://ffmpeg.org/) 打包成**原生 macOS App** 的视频下载小工具，带 **Liquid Glass（液态玻璃）** 界面。无需安装 Python / Homebrew，双击即用。

> 当前版本：**第五版（Liquid Glass 界面版）**
> 适用系统：**macOS 15 Sequoia**（界面用 `NSVisualEffectView` 磨砂玻璃还原，升级到 macOS 26 Tahoe 后观感更原生）

---

## 功能特点

- **原生 Mac App**：Platypus droplet，不是网页、不是命令行脚本
- **开箱即用**：内置 yt-dlp 与 ffmpeg，不依赖本机任何环境
- **Liquid Glass 界面**：无边框悬浮面板 + 桌面实时模糊 + 圆角 + 高光描边（五个交互窗口全部玻璃化：链接输入 / 类型选择 / 进度 / 结果 / 提示）
- **输出 MP4**：自动 remux，不重新编码、不丢画质
- **多链接批量**：一次粘贴多个链接（换行或空格分隔）
- **一键打开**：下载完成的结果窗带「打开下载文件夹」按钮
- **默认保存**：「下载」文件夹，文件名自动用视频标题

---

## 支持的网站（实测）

| 平台 | 视频下载 | 图片（无水印） | 说明 |
| --- | --- | --- | --- |
| 哔哩哔哩 B站 | ✅ | — | 直接可用 |
| YouTube | ✅ | — | 需开启系统代理（HTTP/HTTPS `127.0.0.1:33210`） |
| X / Twitter | ✅ | — | 需开启系统代理 |
| 小红书 | ⚠️ | ❌ | 视频需用 App 内「复制链接」得到的**带 `xsec_token` 的分享链接**；图片无水印暂不支持 |
| 抖音 | ❌ | — | 需 Chrome 登录 douyin.com（当前未集成） |
| 快手 | ❌ | — | yt-dlp 暂无快手提取器 |
| Instagram | — | ❌ | 需 Chrome 登录 + 代理（当前未集成） |

> 说明：YouTube / X 走的是你本机已有的系统代理；B站直连即可。抖音 / 快手 / Ins 属于提取器或登录态限制，并非下载引擎故障。

---

## 安装

1. 在本仓库 **Releases** 页面下载 `视频下载-第五版.dmg`
2. 双击挂载，把 `视频下载.app` 拖到「应用程序」文件夹
3. 首次打开：在「应用程序」里**右键 → 打开 → 仍要打开**
   - 因为 App 是 ad-hoc 自签名，Gatekeeper 会报「无法验证"/已损坏"」，这是正常的，右键「仍要打开」即可放行

---

## 使用

1. 打开「视频下载」App
2. 在输入框**粘贴视频链接**（可一次贴多个，用换行或空格分开）
3. 选择**下载类型**（见下表），点击「开始」
4. 等待进度窗跑完，结果窗会显示文件名，点「打开下载文件夹」即可定位文件

### 下载类型说明

| 选项 | 用途 |
| --- | --- |
| MP4 通用兼容（推荐·进剪辑软件） | 最稳的 MP4，剪映 / FCP / Premiere 都能直接吃 |
| MOV（Final Cut / 剪映） | 给苹果系剪辑软件用 |
| MP4 最高画质（4K/60帧，画质优先） | 原画优先，体积大 |
| 只要音频（MP3） | 只抽音轨 |
| 先列出所有画质让我选 | 弹窗列出可选分辨率，手动挑 |
| 带 Chrome 登录态下载（会员/高清） | 借用 Chrome 的登录 Cookie 下会员/高清内容 |

---

## 常见问题 / 注意事项

- **YouTube、X 下不动？** 确认系统代理已开（HTTP + HTTPS 都指向 `127.0.0.1:33210`）。
- **小红书报链接错误？** 用 App / 网页里「复制链接」拿到的那种带 `xsec_token` 的分享链接，普通浏览器地址栏链接不行。
- **Dock 图标显示异常（满铺/过大）？** 是 macOS 图标缓存没刷新。终端执行：
  ```bash
  sudo rm -rf /Library/Caches/com.apple.iconservices.store
  sudo find /private/var/folders -name 'com.apple.iconservices*' -exec rm -rf {} +
  killall Dock
  ```
  （执行后稍等几秒，图标会回到 80% 内缩版）
- **安全提示**：本 App 仅为个人使用便利打包，下载内容请遵守各平台服务条款与当地法规。

---

## 技术说明 / 构建

- 形态：**Platypus droplet** —— `视频下载.app/Contents/Resources/Scripts/main.scpt`（编译型 AppleScript 驱动）+ `Contents/Resources/bin/`（自带 `yt-dlp`、`ffmpeg`）
- 界面：用 AppleScriptObjC 调用 `NSVisualEffectView`（HUD 材质 + 背后桌面模糊 + 圆角 + 高光描边）实现 Liquid Glass 观感；每个玻璃窗口都带原生弹窗兜底，渲染失败也不会崩界面
- 下载引擎：命令拼接（`--remux-video mp4`）、进度轮询、路径提取、访达定位均保持稳定，第五版只重写了 UI 层
- 图标：应用图标为 80% 内缩版（每边 10% 留白），避免 Dock 上满铺过大

### 版本记录

- **第五版**：Liquid Glass 玻璃界面重写（本文档对应版本）
- 早期版本：修复小红书 `-1700` 报错、M4S→MP4 强制转封装、Dock 图标尺寸等

---

## 许可证

自用工具，随版本分发。yt-dlp 与 ffmpeg 各自遵循其上游开源协议。
