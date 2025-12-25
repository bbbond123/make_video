# AI 视频批量生产方案对标与可复用思路（调研汇总）

## 1. 概览
- 目标：文本驱动 → 每天稳定产出 3 条 AI 视频（30–60 秒），全平台通用。
- 当前方案：n8n 调度 + 文案 → TTS → 视频生成 → 字幕/合成 → 自动发布。
- 结论：该方案与行业常见路径高度一致；可分“稳定产线”和“品质探索”双轨推进。

## 2. 一体化流水线（文本→视频→发布）
- 常见组合：n8n/Make/Zapier + 视频 API（Sora/HeyGen/D-ID/Runway/Pika）+ 云盘/对象存储 + 上传脚本；国内组合常见为 n8n + 飞书多维表格/石墨 + FFmpeg/剪映 + social-auto-upload。
- 实践要点：以表格/JSON 为任务队列；脚本 CLI 化；按阶段产物落盘（input/voice/videos/subtitles/output），便于排错与重试。

## 3. 文本→视频引擎（三类）
- 口播型（Talking Head/Avatar）：HeyGen、D-ID、Synthesia、Elai、Colossyan、Movio。优点：角色一致、口型同步、支持批量/API；适合“AI 真人口播”。
- 场景/镜头型（生成式视频）：Sora/Sora2、Runway、Pika、Haiper。优点：镜头语言强；缺点：时延/成本/不确定性较高，口型弱；适合高光内容，分段拼接。
- 模板拼装型（稳定量产）：FFmpeg + MoviePy + AE/PR 脚本（ExtendScript/Panel），或 CapCut/Canva Web + RPA。优点：稳定、成本低、易控风格；适合“定格+字幕”。

## 4. TTS 与字幕
- TTS：低成本/免费 Edge-TTS；本地 CosyVoice、GPT-SoVITS；商用 阿里云通义TTS、Azure TTS、百度TTS、ElevenLabs。建议：先用 Edge-TTS 起步，预留切换接口。
- 字幕：直接用 TTS 文本生成 SRT/ASS；样式用 FFmpeg drawtext、ASS/SSA、MoviePy；复杂样式可用剪映模板或 PR 脚本。

## 5. 自动化与调度
- 工具：n8n（推荐）、Make、Zapier；或 Python + Celery/cron。
- 模式：飞书/Sheets 作为任务表；n8n 定时扫描 + 执行 + 记录状态。
- 关键点：可重试、超时、告警、幂等（以文件与元数据为准）。

## 6. 角色一致性与 Cameo 类能力
- SaaS：HeyGen/Synthesia 的“品牌数字人/头像库”。
- 本地/开源：SadTalker、Wav2Lip、EMO 等，固定驱动图像实现“定格口播”。
- 生成式一致性：角色设定 + 固定提示词 + 控制图（ControlNet、IP-Adapter）；或使用 Cameo/角色引用参数。
- 中转：国内常见通过 API 中转服务获取海外能力，含计费与额度管理。

## 7. 多平台自动发布
- 脚本库：social-auto-upload（抖音/小红书/视频号/快手/B站/百家号/TikTok）。
- 官方 API：YouTube/TikTok 较完善；国内多依赖 Web 自动化（Playwright/Puppeteer）。
- 集成：n8n 触发 CLI/HTTP，维护 Cookie/代理与定时上线。

## 8. 目录与工程化最佳实践
- 建议结构：
  - scripts/: generate_voice.py, generate_video.py, generate_subtitle.py, compose.py, upload.py
  - templates/: 字幕样式、画面布局、PR/AE 项目文件
  - assets/: 头像底图、背景、BGM、转场
  - input/: 文案、任务表（CSV/JSON）
  - output/: 中间产物与最终视频（含元数据 JSON）
  - config/: 平台密钥、参数（tts.json、render.json、upload.json）
- 工程化要点：脚本参数化；统一日志与元数据；任务号贯穿全链路；支持 resume 与重试。

## 9. 与本方案的对照与建议
- 双轨推进：
  - 稳定轨（1–2 天）：Edge-TTS + 定格/固定头像 + FFmpeg 模板合成 + social-auto-upload。
  - 品质轨（并行）：Sora2/Runway/Pika 分段生成 + 拼接 + 人工挑选。
- 人物一致性：初期固定头像 + SadTalker/Wav2Lip；进阶使用 SaaS 数字人或 Cameo/角色库。
- 成本与质量：优先时延与可预期性，质量通过模板、音乐/字幕样式迭代。

## 10. 检索与验证清单（联网后执行）
- 一体化：n8n video pipeline, n8n text to video, n8n youtube upload, 飞书 多维表格 n8n 视频
- 口播/头像：HeyGen API batch, D-ID API talking head, Synthesia API workflow
- 生成式视频：Runway gen-3 API workflow, Pika API pipeline, Sora n8n template
- TTS：edge-tts bulk, 阿里云 通义TTS API 示例, ElevenLabs batch TTS
- 字幕/合成：FFmpeg subtitle template, MoviePy captions style, 剪映 批量 模板 自动化
- 自动发布：social-auto-upload n8n, Playwright 抖音 上传 脚本
- 一致性：SadTalker batch, Wav2Lip python cli, EMO talking head pipeline

## 11. 风险与规避
- 账号与风控：管理 Cookie/UA/代理，规避异常登录与上传限流。
- 文案版权与合规：避免版权文本/音乐，生成素材注意授权与平台规范。
- 成本波动：生成式视频计费不稳定，设置预算阈值与队列控制。
- 时延与失败重试：长任务需超时与重试策略，异步化处理与状态追踪。

