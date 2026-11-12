# khan-explainer

一个面向 **Windows 10/11** 的 Claude Code 技能：把文字说明渲染成可汗学院风格的手写讲解视频。你描述主题，Claude 生成场景文件，`render.js` 输出 MP4。

这个 fork 的主要开发和验证环境是 Windows。macOS 和 Linux 的代码路径仍保留，但本仓库不把它们作为当前发布版本的主要支持目标，也不承诺与 Windows 版本完全一致。

## 能做什么

- **黑板/白板讲解**：边说边画图，笔画和语音自动同步。
- **截图标注教程**：在产品截图上绘制箭头、圆圈和步骤。
- **真实音频剪辑**：把视频或播客的真实声音配到手绘板书上。
- 横屏、竖屏、深色黑板和浅色白板均可使用。

## Windows 安装

需要：

- Windows 10 或 Windows 11
- Node.js 20 或更高版本
- ffmpeg / ffprobe
- Playwright Chromium
- 至少一个 Windows 语音（系统通过 PowerShell `System.Speech` 调用）

PowerShell：

```powershell
winget install Gyan.FFmpeg
winget install eSpeak-NG.eSpeak-NG
winget install astral-sh.uv
```

然后安装项目依赖：

```powershell
git clone https://github.com/WENGENG-boop/khan-explainer-open-source $HOME\.claude\skills\khan-explainer
cd $HOME\.claude\skills\khan-explainer\scripts
npm ci
npx playwright install chromium
```

安装 Windows 语音：打开“设置 > 时间和语言 > 语音”，添加需要的语言。英文和中文场景优先使用已安装的 Windows 语音；没有对应语言时，渲染器会尝试使用 eSpeak NG。

## 使用

让 Claude Code 生成场景，或直接渲染示例：

```powershell
cd $HOME\.claude\skills\khan-explainer
node scripts/render.js examples/ai-agents.js agents.mp4
$env:KHAN_THEME='light'; node scripts/render.js examples/ai-agents.js light.mp4
Remove-Item Env:KHAN_THEME
node scripts/render.js examples/ui-tutorial.js tutorial.mp4
node scripts/render.js examples/languages.js languages.mp4
```

每次渲染都会输出一个 contact sheet 路径，用来检查每个 beat 结束时的画面。最终 MP4 和 contact sheet 不会自动提交到 Git；`.gitignore` 已排除常见输出、缓存和本地环境目录。

## 语音模式

默认使用 Windows 已安装的系统语音。也可以使用：

- **Kokoro**：本地语音，需要 Python 3.12 和 `uv`；第一次运行会下载模型。
- **ElevenLabs**：可选的在线语音，需要你自己设置 `ELEVENLABS_API_KEY`，不要把 key 写入场景文件或提交到 Git。
- **真实音频**：场景使用 `audio` 和 beat 时间戳。

Kokoro 示例：

```powershell
cd $HOME\.claude\skills\khan-explainer\scripts
uv venv --python 3.12 .venv
uv pip install --python .venv kokoro soundfile
$env:KHAN_TTS='kokoro'
node ..\tests\run.js out
Remove-Item Env:KHAN_TTS
```

真实音频剪辑和 talking-head 示例还需要 `uv` 以及相应的本地依赖；Windows 下的 `talking_head_example.sh` 通过 Git Bash 运行，人物抠像使用 MediaPipe。macOS Vision 代码是保留的兼容路径，不是本 fork 的主要验证路径。

## 测试

在 PowerShell 中：

```powershell
cd $HOME\.claude\skills\khan-explainer
node tests/fonts.js
node tests/run.js out
```

`tests/run.js` 会渲染示例和多语言场景，并在出现布局警告时失败。仓库中的 `tests/results/SUMMARY.md` 保存了此前的测试记录；它不是当前机器运行结果的保证。

## 项目结构

- `scripts/render.js`：Windows 语音、Playwright、ffmpeg 渲染入口。
- `scripts/board.html`：浏览器画板和绘图 API。
- `examples/`：示例场景。
- `tests/`：字体和多语言渲染测试。
- `SKILL.md`：完整场景 API 和使用说明。

## 许可证

MIT。项目采用 “Khan Academy style” 的视觉描述，与 Khan Academy 官方没有隶属或背书关系。
