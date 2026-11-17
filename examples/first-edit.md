# 第一次用 ffmpeg-skill 演练 | First ffmpeg-skill drill: probe, cut, verify

拿一条你自己的横屏素材把关键动作走一遍，先量后剪，剪完自验再交付：

1. **装与体检**：跑 `npx ffmpeg-skill` 装好，再跑 `npx ffmpeg-skill doctor` 体检本机组件。总检转红先别慌——逐件看 `doctor --json` 的 `tools` 字段，确认你要用的那几件是 usable；总 ok 红往往只是一件工具缺一个滤镜（例如 Homebrew 版 ffmpeg 缺 subtitles）。
2. **先量**：对素材跑 probe，把时长、帧率、分辨率、色彩与音轨布局看清再动手。后面所有参数都照这份实测填，不照文件名猜。
3. **粗剪**：先用去静音把口播的空白压掉（`silence.py` 按阈值处理），再按内容剪接。能无损就不重编码，这一步先把时长降下来。
4. **改画幅出竖屏版**：用 `fit.py --aspect 9:16 --fit crop` 把横屏裁成竖屏，或直接走平台模板：`render.py --template tiktok` 一条命令出 TikTok 版；要七个视频平台一次出齐就 `--template all`。
5. **响度与字幕**：出片前走一遍响度归一（`loudness.py` 按目标平台口径）；字幕按原文烧，位置走模板的安全区。要语音转写字幕，先单独装本地引擎（`pip install faster-whisper` 最简单），首次运行会下模型权重，音频不出本机。
6. **自验再交付**：跑交付检查对一遍目标平台规格，再抽接触表亲眼看过画面——时长量对了不等于画面看对了。两样都过了才报完成；检查里标 WARN 的平台上限项是参考口径，上传前再对一眼平台当下规则。
