# 下一步与可落地清单（讨论/验证阶段）

## 1) 目标与阶段性验收
- 近期目标：在本地以“定格+字幕”路线跑通单条视频端到端（文案→TTS→合成→导出）。
- 阶段验收：
  - 输入 `input/texts/demo.txt`，输出 `output/videos/demo.mp4`。
  - 产出 `output/meta/demo.json`，包含各环节耗时/状态。
  - 可重复执行，失败可重试，幂等。

## 2) 可选实现路径（多路线并行探索）
- 路线A：模板合成（稳定产线优先）
  - Edge-TTS 生成旁白 → FFmpeg/MoviePy 合成固定模板 → 生成 9:16 视频。
  - 优点：成本低、可控；缺点：镜头变化弱。
- 路线B：开源口播（一致性优先）
  - 固定人像 + SadTalker/Wav2Lip + Edge-TTS → 口播视频 → 加字幕样式。
  - 优点：人物稳定；缺点：环境/模型准备成本。
- 路线C：生成式片段（品质探索）
  - Sora2/Runway/Pika 生成 8–12s 片段 ×N → FFmpeg 拼接 → 配字幕/旁白。
  - 优点：镜头强；缺点：成本与时延，口型弱。
- 路线D：SaaS 数字人（口播快速验证）
  - HeyGen/D-ID/Synthesia API → 生成口播 → 加品牌模板/字幕 → 导出。
  - 优点：省心；缺点：按条计费且受平台限制。

## 3) 初始脚本骨架（占位设计）
- scripts/generate_voice.py
  - 功能：Edge-TTS 生成 wav/mp3；输入文本或 JSON；可选发音人/语速。
  - 参数：`--text_file` `--out` `--voice` `--rate` `--pitch`。
  - 输出：音频文件 + `meta`（时长、采样率）。
- scripts/generate_subtitle.py
  - 功能：由文本或 TTS 时间线生成 SRT/ASS；可设最大行长与每行字数。
  - 参数：`--text_file` `--audio` `--out` `--style`。
  - 输出：`.srt` 或 `.ass`，并写入样式说明。
- scripts/compose.py
  - 功能：FFmpeg/MoviePy 将背景、人物图、TTS 音频、字幕合成 9:16。
  - 参数：`--bg` `--portrait` `--audio` `--subtitle` `--template` `--out`。
  - 输出：`.mp4`（h264+aac），支持码率/帧率配置。
- scripts/generate_video.py
  - 功能：封装外部 API（Sora/Runway/HeyGen 等）的调用与回调轮询。
  - 参数：`--provider` `--prompt` `--duration` `--out`。
  - 输出：片段或成片；失败重试与超时控制。
- scripts/upload.py
  - 功能：调用 social-auto-upload 或自建 Playwright 脚本上传。
  - 参数：`--platform` `--file` `--title` `--desc` `--schedule`。

## 4) 元数据与任务格式（草案）
- 任务 JSON（input/tasks/*.jsonl）
  - 字段：`id,title,text,voice,template,status,retry,budget`
- 产出元数据（output/meta/{id}.json）
  - 字段：`id,steps:[{name,status,duration_ms,outputs}],cost,created_at`。

## 5) n8n 工作流示意（无代码到半自动）
- Workflow-1 提交任务：
  - 触发：表格/HTTP → 写入 `input/tasks/` → 触发队列。
- Workflow-2 周期执行：
  - 读取新任务 → 调 `generate_voice.py` → `generate_subtitle.py` → `compose.py` → 写入 meta → 通知。
- Workflow-3 自动上传（可选）：
  - 成片→ 调用 `upload.py`（social-auto-upload）→ 写入上传状态。

## 6) 依赖与环境建议
- Python 3.10+，ffmpeg 可执行文件在 PATH。
- 可选：node + playwright（自动上传），Docker 化 n8n。
- `.env` 管理第三方密钥与代理，参考 `config/.env.example`。

## 7) 验证与测试用例
- 单元级：
  - TTS：同一文本、不同发音人/语速 → 检查时长/采样率。
  - 字幕：中英/标点文本 → 行长与断句稳定性测试。
  - 合成：不同分辨率/码率 → 输出兼容性（微信/抖音）。
- 流水线级：
  - 空文本/超长文本 → 失败与提示。
  - 重复任务 id → 幂等与覆盖策略。

## 8) 风险与替代项
- 上传风控：
  - 方案1：social-auto-upload；方案2：Playwright 自研；方案3：仅导出，人工复核发布。
- 人物一致性：
  - 方案1：固定头像 + SadTalker/Wav2Lip；方案2：SaaS 数字人；方案3：Cameo/角色库。
- 成本控制：
  - 方案1：以模板合成为主；方案2：生成式片段限额；方案3：批处理+人工抽检。

## 9) 下一步可执行 TODO（按优先级）
- P0：提供 demo 文案，添加到 `input/texts/demo.txt`。
- P0：实现 Edge-TTS 与 FFmpeg 的最小可用脚本骨架（仅参数解析+占位日志）。
- P1：定义 `output/meta` 的 JSON 结构与写出逻辑。
- P1：草拟 n8n 工作流节点与命令节点示意图（markdown）。
- P2：准备一套基础模板与素材（背景、字体、颜色）。

---

附：如需，我可继续提交脚本骨架与 `.env` 示例填充，方便你本地快速起步。
