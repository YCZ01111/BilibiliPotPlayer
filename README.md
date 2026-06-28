# BilibiliPotPlayer

适用于 PotPlayer 的 Bilibili 插件。如果配合[油猴脚本](https://greasyfork.org/zh-CN/scripts/461800-bilibilipotplayer)，可以直接在网页打开 PotPlayer 进行播放

## 与原版的区别

本 fork 基于 [chen310/BilibiliPotPlayer](https://github.com/chen310/BilibiliPotPlayer) 改造，主要针对新版 PotPlayer（260114+）无法播放 B 站视频的问题：

| 对比项 | 原版 | 本 fork |
|---|---|---|
| 播放解析方式 | 直接请求 B 站 API，伪造 itag 让 PotPlayer 配对 DASH 音视频流 | 调用 [yt-dlp](https://github.com/yt-dlp/yt-dlp) 解析 B 站流，用 `HostGetITag` 动态申请有效 itag |
| 新版 PotPlayer 兼容性 | 260114+ 完全无法播放（itag 伪造机制被移除） | 正常播放 |
| 额外依赖 | 无 | 需要安装 yt-dlp.exe |
| 登录态 | 配置文件填写 Cookie 字符串 | Cookie 字符串 +（可选）Netscape 格式 cookie 文件 |
| 浏览/搜索/番剧列表 | 支持 | 支持（未改动） |
| 默认画质 | 由 B 站 API 返回顺序决定 | 默认为 yt-dlp 解析到的首个视频流（通常非最高画质，可在右下角画质菜单手动切换） |

**改造原因**：新版 PotPlayer 移除了通过伪造 itag 配对 DASH 音视频流的内部机制，而 B 站现代内容只返回 DASH（m4s 分离流）不返回 MP4 单文件，导致原版完全无法播放。yt-dlp 内置 bilibili extractor，能正确处理 WBI 签名、DASH 音视频配对和防盗链。

## 使用说明

### 前置要求

- PotPlayer **260114 或更高版本**（提供 `HostExecuteProgram`/`HostGetITag` 等扩展 API）
- [yt-dlp.exe](https://github.com/yt-dlp/yt-dlp/releases/latest)（放到 PotPlayer 的 `Module` 目录，或在配置文件指定路径）
- （可选）Netscape 格式的 B 站 cookie 文件（播放会员/番剧内容需要）

### 快速开始

1. 按下方「安装插件」和「安装 yt-dlp」章节完成安装
2. 在 `Bilibili_Config.json` 中配置 cookie 和 cookie 文件路径
3. 重启 PotPlayer
4. 按 <kbd>ctrl</kbd> + <kbd>U</kbd> 粘贴 B 站视频/番剧链接，或拖拽链接到 PotPlayer 窗口
5. 播放后可在 PotPlayer 右下角画质菜单切换清晰度

### 调试

在 `Bilibili_Config.json` 中设置 `"debug": true`，然后打开 PotPlayer 控制台（视图 → 控制台），可看到 yt-dlp 的调用日志，包含：
- `yt-dlp cmd:` 实际执行的命令行
- `yt-dlp: bestVideo=xxxp bestAudio=xxxK` 解析到的最高画质

### 配置文件字段说明

`Bilibili_Config.json` 新增字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `useYtDlp` | bool | 是否使用 yt-dlp 解析播放。`true`（默认）= 使用 yt-dlp；`false` = 回退到原版逻辑（新版 PotPlayer 上无法播放） |
| `ytdlpPath` | string | yt-dlp.exe 的完整路径。留空则自动查找 `{PotPlayer安装目录}\Module\yt-dlp.exe` |
| `cookieFile` | string | Netscape 格式 cookie 文件路径，用于 yt-dlp 登录态。可用浏览器扩展 [Get cookies.txt](https://chromewebstore.google.com/detail/get-cookiestxt/bgaddhkoddajcdgocglldofjekigamnk) 或 [Cookie-Editor](https://cookie-editor.cgagnier.ca/) 导出 |

## 安装插件

[下载项目](https://github.com/chen310/BilibiliPotPlayer/archive/refs/heads/master.zip)

将项目 `Media/PlayParse` 路径下的 `MediaPlayParse - Bilibili.as`、`MediaPlayParse - Bilibili.ico` 和 `Bilibili_Config.json` 三个文件复制到 `{PotPlayer 安装路径}\Extension\Media\PlayParse` 文件夹下。

`MediaPlayParse - Bilibili.as` 提供了解析 `Bilibili` 链接的功能。

将项目 `Media/UrlList` 路径下的 `MediaUrlList - Bilibili.as` 和 `MediaUrlList - Bilibili.ico` 两个文件复制到 `{PotPlayer 安装路径}\Extension\Media\UrlList` 文件夹下。

`MediaUrlList - Bilibili.as` 提供了列出 `Bilibili` 常用链接的功能，使用方式为按下 <kbd>ctrl</kbd> + <kbd>U</kbd> 并选择要播放的项目，如下图所示

![UrlList](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/urllist.png)

## 安装 yt-dlp（PotPlayer 260114+ 必需）

新版 PotPlayer 移除了通过伪造 itag 配对 DASH 音视频流的内部机制，导致 B 站视频（音视频分离的 m4s 格式）无法播放。本插件已改造为调用 [yt-dlp](https://github.com/yt-dlp/yt-dlp) 解析 B 站流，配合 PotPlayer 新增的 `HostGetITag` 等 API 动态申请有效 itag。

### 安装步骤

1. **下载 yt-dlp.exe**

   前往 [yt-dlp releases](https://github.com/yt-dlp/yt-dlp/releases/latest) 下载 `yt-dlp.exe`（Windows 64-bit 版本）。

2. **放置到 PotPlayer 目录**

   将 `yt-dlp.exe` 放到 PotPlayer 安装目录下的 `Module` 文件夹中，完整路径示例：

   ```
   C:\Program Files\DAUM\PotPlayer\Module\yt-dlp.exe
   ```

   如果 `Module` 文件夹不存在，手动创建即可。

   也可以在配置文件 `Bilibili_Config.json` 中通过 `ytdlpPath` 字段指定任意路径。

3. **（可选）配置 Cookie 文件**

   播放会员内容/番剧需要登录态。yt-dlp 使用 Netscape 格式的 cookie 文件，可用以下任一浏览器扩展导出：

   - **Cookie-Editor**（推荐，Edge/Chrome/Firefox 均可用）：登录 B 站后点击扩展图标 → 右下角 **Export** → 选择 **Netscape** 格式
   - **Get cookies.txt**：登录 B 站后点击扩展图标导出

   将导出的文件保存为 `bilibili_cookies.txt`，然后在配置文件 `Bilibili_Config.json` 中设置 `cookieFile` 为该文件的完整路径。

4. **重启 PotPlayer**

   修改完配置文件后需要重启 PotPlayer 才能生效。

## 登录

找到刚刚复制过去的配置文件 `Bilibili_Config.json`，填写 Cookie 等设置内容。 打开 PotPlayer，按 <kbd>F5</kbd> 打开选项，点击`扩展功能`下的`媒体播放列表/项目`，再点击 `Bilibili`，然后打开`账户设置`，填写配置文件路径，如 `D:\DAUM\PotPlayer\Extension\Media\PlayParse\Bilibili_Config.json`。每次修改完配置文件，可能都要重启 PotPlayer 才能生效。

![Settings](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/settings.png)

点击测试按钮，如果弹出账号信息，就说明登录成功。

![Test](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/test.png)

### Cookies 获取

[获取Cookie](https://github.com/chen310/BilibiliPotPlayer/issues/62#issuecomment-1841909583)

## 使用方法

### 播放视频/直播

将 Bilibili 链接拖到 PotPlayer，或者按 <kbd>ctrl</kbd> + <kbd>U</kbd> 粘贴 Bilibili 链接即可播放。可参考[视频](https://www.bilibili.com/video/BV1mM41177kT)

### 搜索

按 <kbd>ctrl</kbd> + <kbd>U</kbd>，在文件地址列表中选择`搜索`，然后到上面的输入框中替换关键词，最后回车即可

![Search](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/search.png)

### 跳过片头片尾

对于一些电视剧、番剧，能够跳过片头和片尾。具体设置为：在 PotPlayer 上点击鼠标右键，选择`播放`-`跳略播放`-`跳略播放设置`

![Skip_Settings](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/skip_1.png)

勾选`跳略播放`和`章节名称`，并在名称列表中追加`哔哩哔哩-片头`和`哔哩哔哩-片尾`两项，每一项之间用英文分号`;`隔开。

![Skip_Settings](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/skip_2.png)

### 在列表中显示缩略图

按 <kbd>F6</kbd> 打开播放列表，点击鼠标右键，点击`样式`，选择`显示缩略图`，即可显示视频的缩略图。

![Thumbnail](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/thumbnail.png)

### 创建自动更新的播放列表

按 <kbd>F6</kbd> 打开播放列表，点击新建专辑，起一个合适的专辑名称，选择外部播放列表，并填写相应的链接，再点击确定即可。这样就得到一个可以自动更新的列表。

![Create_Playlist](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/create_playlist_1.png)

![Create_Playlist](https://cdn.jsdelivr.net/gh/chen310/BilibiliPotPlayer/public/create_playlist_2.png)
