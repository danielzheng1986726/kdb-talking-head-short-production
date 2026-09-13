# 本机适配：Claude Code（模板）

上游 Skill 面向 Codex，本文件是 Claude Code 在 macOS 上的适配层。上游 SKILL.md 与 references/ 保持原样，方便 `git pull` 同步；本机差异只写在这里。

## 调用方式

在 `~/.claude/CLAUDE.md` 的口令表里加一行「做视频」+ 路径 → 走本 skill，就能用中文触发。

上游 SKILL.md 只动了两处，`git pull` 冲突时保留即可：① frontmatter description 末尾补的中文触发语；② 「Finish the media correctly」段开头指向本文件的一句。

Claude Code 里直接说「用 kdb-talking-head-short-production 处理这条视频」，或给视频路径 + 需求。不需要 `$` 前缀，`agents/openai.yaml` 对 Claude Code 无效可忽略。

## 本机工具栈（需要自己验证）

| 角色 | 实现 | 说明 |
|---|---|---|
| 探测 / 编码 | FFmpeg（含 libass、libx264） | `brew install ffmpeg` 或系统自带 |
| 本地中文 ASR + 词级对齐 | mlx-qwen3-asr，Qwen3-ASR-1.7B + Qwen3-ForcedAligner-0.6B | Apple Silicon 本地推理，模型首次运行自动下载 |
| 热词 | `--context "你的常用专有名词"` | 软偏置，不是硬词表；按你的内容领域填 |
| 字幕字体 | PingFang SC（macOS 系统资产） | fontconfig 看不到，需显式给 fontsdir（见下方坑） |
| 备用 ASR | openai-whisper / mlx_whisper | 仅当主 ASR 出幻觉需交叉核对时用 |

**ASR 命令示例：**

```bash
mlx-qwen3-asr --model Qwen/Qwen3-ASR-1.7B --language Chinese --timestamps -f all -o <输出目录> <16k.wav>
```

**PingFang SC 字体路径：** macOS 的 PingFang 放在 `/System/Library/AssetsV2/com_apple_MobileAsset_Font8/` 下，具体子目录因系统版本而异。用以下命令找到：

```bash
find /System/Library/AssetsV2/com_apple_MobileAsset_Font8 -name "PingFang*" -type f 2>/dev/null | head -5
```

找到后在 ffmpeg 的 subtitles filter 里用 `fontsdir=<那个目录>`。

**已知坑：**
- Bash 工具跑的是 zsh，filter 字符串里 `$s:end=` 会被当成 `$s:e` 修饰符。用 Python 生成 filter 并 `-filter_complex_script` 读文件。
- 1.7B 在开头几秒和长静音处偶发幻觉；对照 silencedetect 的静音区间看 ASR 有没有在空档里编话。
- M1/M2/M3 Air 无风扇：90s 素材约 30s 转完，不必用更小模型。

## 你的偏好（按自己情况改）

这里写你对视频的固定偏好，Agent 每次都会读到。以下是模板，按实际情况改：

- **不重录。** 只做现有素材，建议留给 NEXT_TIME.md。
- **保留自然表达。** 弃用的开头和支线可以删；句中口癖不做音节手术，只在显示字幕里去掉。
- **硬切不遮盖。** 走路自拍的跳切落在停顿上即可，不加转场、不做动效。
- **字幕先行，图解按需。** 如果口播已经讲清楚了，可以没有任何图解。
- **封面 = 首帧上的两行大字。** 主题一行 + 判断一行。PingFang SC 粗体，标题约 128–140px，字幕 86px，白字黑边 5px，底边距 390（避开平台底部操作栏）。
- **发布自己点。** Agent 只交 MP4 / 封面 / SRT / 文案草稿，不上传。

## 工程结构（默认）

```
~/Projects/active/talking-head-shorts/<YYYY-MM-DD>-<slug>/
  source/PROVENANCE.txt          # 原文件路径 + sha256（不复制 100MB+ 原片）
  work/inspect/                  # ffprobe 记录、contact sheet、QA 帧
  work/transcript/               # audio_16k.wav、raw_<model>/、corrected_transcript.md
  work/process/                  # segments.json、build_captions.py、timeline_map.json、final.ass/.srt、aroll_raw.mov
  output/                        # *.ready-to-upload.mp4、*.cover.jpg、*.srt、QA.md、NEXT_TIME.md、POST_DRAFT.md
```

`work/process/build_captions.py` 是可复用的：改 `segments.json` 和 CUES 表即可重建 ASS/SRT/时间映射。

## 交付流程

1. ffprobe 全量探测 + contact sheet + 首/中/尾帧；抽 16k 单声道 wav。
2. 1.7B 转写（`-f all` 拿 json 词级时间）；silencedetect 找停顿。
3. 读全稿，写 content_map.md（promise ledger + 删除表 + 保留段）。
4. Python 生成 trim/concat filter → 干净 A-roll（crf 16，pcm 音频），loudnorm 第一遍测量。
5. build_captions.py：词级时间 → 短句 cue → 通过保留段映射到成片时间线 → ASS（标题+字幕）+ SRT。
6. 最终编码：A-roll + subtitles + loudnorm 第二遍（I=-16, TP=-1.5），libx264 crf 20，aac 48k 192k，`-map_metadata -1 -map_chapters -1 -movflags +faststart`。
7. QA：解码无错、首尾时长一致、无黑帧、响度、封面与第 0 帧 PSNR 一致、抽标题期/切点前后/最长字幕帧看图。
8. 写 QA.md、NEXT_TIME.md、POST_DRAFT.md。不发布。
