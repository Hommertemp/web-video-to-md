# 网络视频转成 md / Web Video to Markdown

从 B 站、YouTube、微博、西瓜、腾讯视频等站点下载视频或音频，并在本机用 MarkItDown 转写成可检索的 Markdown 文字稿。全程本地处理，媒体文件不上传。

Download online video/audio with `yt-dlp`, then transcribe it locally into searchable Markdown with MarkItDown. Nothing is uploaded — transcription runs on your machine.

## 它做什么

一个视频链接进去，两样东西出来：

- 一份留存用的媒体文件（默认上限 1080p）
- 一份可检索的 Markdown 文字稿

适合：网课、讲座、会议录像、播客、访谈、合集/播放列表批量整理。

## 安装

这是一个 Hermes Agent 技能。把本仓库内容放到技能目录下即可：

```
~/.hermes/skills/productivity/web-video-to-md/
├── SKILL.md
├── scripts/
│   ├── video_to_md.py
│   ├── fetch_videos.py
│   └── _vtmd_common.py
└── references/
    ├── sites.md
    └── troubleshooting.md
```

Windows 下为 `C:\Users\<你>\AppData\Local\hermes\skills\productivity\web-video-to-md\`。

依赖：

```bash
python -m pip install -U yt-dlp
# ffmpeg：Windows 可用 winget install Gyan.FFmpeg
# 转写阶段依赖 MarkItDown 技能（含 Whisper）
```

## 快速开始

以下 `<skill>` 指本技能 SKILL.md 所在目录。

```bash
# 全流程：下载 9 集合集 + 转写成 md
python "<skill>/scripts/video_to_md.py" --url "https://www.bilibili.com/video/BV1bt421578n/" --work-dir "E:/课程/Efficiently-Serving-LLMs"

# 只要文稿：只下音轨，省约 90% 流量
python "<skill>/scripts/video_to_md.py" --url "https://youtu.be/XXXX" --audio-only --work-dir "./out"

# 只下载，先不转写
python "<skill>/scripts/fetch_videos.py" --url "https://www.bilibili.com/video/BV1bt421578n/" --output-dir "E:/视频" --max-height 1080

# 先探路：看看这个链接有几集
python "<skill>/scripts/fetch_videos.py" --url "https://www.bilibili.com/video/BV1bt421578n/" --probe
```

产物布局：

```
<work-dir>/
├── media/                        # 媒体文件 + _video-fetch.log
├── markdown/                     # 转写出的 .md
└── web-video-to-md-receipt.json  # 总回执
```

## 决策表

| 用户实际诉求 | 做法 |
|---|---|
| 只要媒体文件 | `fetch_videos.py`，或 `video_to_md.py --skip-md` |
| 只要文字稿/笔记 | `video_to_md.py --audio-only` |
| 媒体和 md 都要 | `video_to_md.py`（默认） |
| 只想知道合集有几集 | `fetch_videos.py --probe` |
| 媒体已下好，只想重跑转写 | `video_to_md.py --skip-download` |
| 只要其中几集 | 加 `--playlist-items 1-9` 或 `1,3,5` |

## 几个关键点

- **版本保鲜**：yt-dlp 用日期版本号。超过 90 天会被 B 站以 HTTP 412 拒绝，脚本会自动升级再重试；遇到 403/429/风控页也会升级重试一次。要锁版本加 `--no-update`。
- **合集展开**：默认 `--yes-playlist`。B 站"合集"常是单视频多分P（`?p=1..9`），会展开成 9 个文件。
- **需要登录的内容**：先让用户在浏览器登录，再 `--cookies-from-browser chrome:Default`（Edge 写 `edge`）。
- **模型选择**：`--whisper-model base` 为默认；`tiny` 最快，`small` 更准。base 在普通 x86 CPU 上约 12 倍实时。
- **续跑**：中断后用同一 `--work-dir` 加 `--resume`，已完成段落会复用。
- **术语校对**：语音转写会听错产品名、模型名、人名。交付时结合站点字幕（`--sub-langs`）校正。

## 不做什么

- 不做媒体格式转码、压制、剪辑、抽帧
- 不代替网页搜索/研究 —— 只要正文就别下视频
- 不尝试绕过会员/付费内容

## 文档

- [references/sites.md](references/sites.md) — 各站点取流要点
- [references/troubleshooting.md](references/troubleshooting.md) — 故障定位与修复
- [SKILL.md](SKILL.md) — 完整技能说明

## License

MIT
