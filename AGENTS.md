# Agent 安装说明

灰泽满 · 作者：栗帽 · 形象 3.4.0 · 发布修订 r6

## 环境

- Coopanion 0.2.8，API 默认 `http://127.0.0.1:17788`。
- 人设安装：Python 3.10+。
- 可选语音：独立 Python 3.11 环境、NVIDIA CUDA、FFmpeg。

从解压目录执行命令，将以下占位符替换为实际绝对路径：

| 占位符 | 路径 |
|---|---|
| `WORKSPACE` | Coopanion 的 workspace 目录 |
| `VOICE_PYTHON` | 语音环境的 python.exe |
| `GPT_SOVITS_REPO` | GPT-SoVITS 源码目录 |
| `ABSOLUTE_RELEASE` | 本发布包解压目录 |
| `WEB_DIR` | 应用的 resources/app/node_modules/cortico-world-desktop-pet/web |

## 1. 形象与人设

在装扮页导入 `hazel-3.4.0.zip`，选择“灰泽满”。配色切换发型，服装独立选择。聊天空闲时执行：

```powershell
python persona/install.py --workspace "WORKSPACE" --base-url http://127.0.0.1:17788
```

脚本自动备份、安装并重载人设。设定和语料已合并在 `persona/payload/CONSTITUTION.md`。

## 2. ChatGPT 订阅

在模型设置选择 `chatgpt`，完成登录授权，设置模型 `gpt-6.1-sol`、推理强度 `low`，测试成功后启用。

Agent 也可使用本地 API：

1. `GET /api/providers`，复用已有 `chatgpt` 端点；没有则 `POST /api/providers`：

```json
{"name":"chatgpt","entry":{"kind":"chatgpt","baseUrl":"https://api.openai.com/v1","spec":{"model":"gpt-6.1-sol","thinking":true,"reasoningEffort":"low"},"multimodal":true,"pricing":[],"options":{}}}
```

2. `POST /api/webui/providers/llm%3Achatgpt/panels/chatgpt/signIn`，请求体 `{"args":["chatgpt"]}`。打开返回的 URL，完成账号验证与授权。
3. 调用同一路径的 `state` 方法、相同请求体，确认 `signedIn=true`；可用模型见返回列表。
4. `POST /api/providers/chatgpt/test`，请求体 `{}`；返回 `ok=true` 后调用 `POST /api/providers/chatgpt/activate`，请求体 `{}`。

## 3. 本地语音（可选）

配置：GPT-SoVITS v5 Turbo、E 参考音色、1.2 倍速。

准备 GPT-SoVITS 源码（提交见 `SOURCES.md`）、FFmpeg 和独立 Python 3.11 环境，执行：

```powershell
& "VOICE_PYTHON" -m pip install torch==2.7.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/cu128
& "VOICE_PYTHON" -m pip install -r voice/requirements-inference.txt
git -C "GPT_SOVITS_REPO" apply --check "ABSOLUTE_RELEASE/voice/runtime-local-changes.patch"
git -C "GPT_SOVITS_REPO" apply "ABSOLUTE_RELEASE/voice/runtime-local-changes.patch"
```

已应用的补丁跳过。下载 [G2PWModel.zip](https://huggingface.co/XXXXRT/GPT-SoVITS-Pretrained/resolve/main/G2PWModel.zip)，解压到 `GPT_SoVITS/text/G2PWModel`，然后执行：

```powershell
& "VOICE_PYTHON" voice/configure.py --repo "GPT_SOVITS_REPO" --download-models
powershell -NoProfile -File voice/start-api.ps1
```

配置器下载并校验模型，生成 `runtime.local.json`。主模型约 1.82 GB，另需 G2PW 和运行环境空间。

等待 `http://127.0.0.1:9881/health` 返回 `ready=true`，安装桌宠语音适配：

```powershell
& "VOICE_PYTHON" voice/install-desktop.py --web "WEB_DIR"
```

重启 Coopanion，启用“说话声”。试听页面：`http://127.0.0.1:9881/`。服务使用端口 9880、9881。

## 使用与回退

- 启动语音：运行 `voice/start-api.ps1`；停止：运行 `voice/stop-api.ps1`。
- 安装后发送一句日常消息，检查回复、播放、停止和静音。
- 人设恢复：`python persona/restore.py --backup "BACKUP_CONSTITUTION"`。
- 语音适配恢复：`python voice/restore-desktop.py`，然后重启 Coopanion。
- 模型切回：`POST /api/providers/原端点名/activate`，请求体 `{}`。
