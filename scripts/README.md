# scripts

用途：存放各环节的自动化脚本（CLI 化，便于 n8n 调用）。

建议包含：
- generate_voice.py（TTS 语音生成）
- generate_video.py（视频生成/对接 API）
- generate_subtitle.py（字幕生成/样式）
- compose.py（FFmpeg/MoviePy 合成导出）
- upload.py（多平台上传/触发器）

要求：
- 所有脚本接受参数（输入/输出路径、模板名、时长等）。
- 输出产物与元数据 JSON，便于重试与追踪。
