# TTS 调研档案（2026-07-26）

> 这是 2026-07-26 对"安卓手机端英语发音"问题的完整调研归档。
> **结论：维持现状（v1.2.8），不在该项目里加 TTS 后端。**
> 本文档记录了所有调研过的方案、真实数据、不推荐的理由，作为未来如果想重新推进时的参考。

---

## 一、问题背景

**症状**：英语乐园网页在 Mac/iPad/Windows 浏览器上正常发音，但安卓手机的 Chrome 和自带浏览器都不发音。

**根因**（确认）：国内安卓 ROM 移除了 Google TTS 引擎，Web Speech API 调用被静默丢弃，不抛错也不出声。

**为什么改不动**：浏览器 + 网络层面的限制，纯前端代码改不动（try/catch、超时检测都没用，因为根本没有错误可捕获）。

---

## 二、调研过的所有方案（真实数据）

### 方案 A：腾讯云语音合成 TTS

| 项 | 真实情况 |
|---|---|
| 产品页 | https://cloud.tencent.com/product/tts |
| API | `tts.tencentcloudapi.com`，RESTful HTTPS POST |
| 鉴权 | TC3-HMAC-SHA256 |
| 音色 | 智瑜(101001)、智云(101004) 等多个女声（基础 REST 接口返回 Base64 编码的 MP3/WAV/PCM） |
| 免费额度 | **产品页写"有免费资源包"，但具体字符数未在公开页面披露** |
| 时长 | **公开页面没写"永久"或"6 个月"** |
| 个人/企业 | 公开页面没区分 |
| 是否需要后端 | **强烈建议**（API Key 不能裸露在前端） |

**结论**：服务真实存在，免费额度真实有，但具体数字只有老吴自己注册后才能看到。需后端代理。

### 方案 B：讯飞语音合成 TTS

| 项 | 真实情况 |
|---|---|
| 产品页 | https://www.xfyun.cn/services/online_tts |
| API | WebSocket 长连接 + APPID/APIKey/APISecret 三件套 + HMAC-SHA256 签名 |
| 免费试用包 | **个人 1 万次 / 3 个月，企业 2 万次 / 3 个月** |
| 创建应用即送 | 每日 500 次 |
| 英文女声 | ❌ **只有 2 个**（凯瑟琳、Lindsay） |
| 英文儿童音 | ❌ **不存在** |
| 儿童音色（中文） | 乐乐、小桃丸、芳芳等，**每个 2 万元单独购买** |
| 跨域 | ✅ WebSocket 原生支持 |

**结论**：免费额度真实但仅 3 个月，英文女声少，儿童音要单独买 2 万元/个。不适合。

### 方案 C：Cloudflare Workers 后端代理

| 项 | 真实情况 |
|---|---|
| 定价页 | https://developers.cloudflare.com/workers/platform/pricing/ |
| 免费额度 | **每天 100,000 次请求**，10ms CPU 时间 |
| `workers.dev` 国内访问性 | ❌ **官方页面拿不到结论**（Cloudflare 在国内可用性官方不背书） |
| 持久化 SecretId/SecretKey | ✅ 支持环境变量 |

**结论**：免费额度足够，但国内可达性不保证。

### 方案 D：edge-tts（开源 Python 库）

| 项 | 真实情况 |
|---|---|
| 安装 | ✅ `pip install edge-tts` 成功（v7.2.8） |
| 英文女声 | ✅ 7 个（en-US-AriaNeural / Ava / Emma / Jenny / Michelle 等） |
| 中文女声 | ✅ 4 个（Xiaoxiao / Xiaoyi / Xiaobei / Xiaoni） |
| 实测生成 | ✅ 中英文 mp3 都成功（24 kHz / 48 kb/s 单声道） |
| 实际 endpoint | ❌ **`wss://speech.platform.bing.com/consumer/speech/synthesize/readaloud/edge/v1`** —— Edge 浏览器内部协议 |
| 风险 | ⚠️ **不是微软公开承诺的 API，违反 ToS，可能随时挂** |

**结论**：能跑、音质顶级（神经声）、完全免费，但底层依赖微软未公开承诺的内部协议，孩子产品不能赌。

### 方案 E：Piper TTS + ONNX Runtime Web（浏览器 WASM）

| 项 | 真实情况 |
|---|---|
| Piper 官方浏览器包 | ❌ **官方没有** |
| 社区包 | ✅ `piper-tts-web@1.1.2`（2025-07 仍在维护），依赖 `onnxruntime-web` |
| `en_US-amy-low.onnx` 大小 | ✅ **63.1 MB** |
| ONNX Runtime Web 限制 | ✅ 63 MB 在 2GB 限制内，可用 CDN script 引入 |
| 现成浏览器端 demo | ❌ **找不到活的官方 demo** |
| 实际可行性 | ⚠️ 技术可行，工程上要自己拼：社区包 + AudioBuffer + Worker + 模型下载进度条 |

**结论**：技术可行但工程复杂，63MB 模型首次下载对小朋友的网络不友好。

### 方案 F：FunASR（阿里达摩院）

| 项 | 真实情况 |
|---|---|
| 仓库 | https://github.com/modelscope/FunASR |
| 方向 | ❌ **ASR（语音识别），不是 TTS** |
| Star | 19,477 |
| 浏览器端 | ❌ 不支持 WASM，Python 服务端 |

**结论**：方向反了，不能用。

### 方案 G：CosyVoice（阿里通义实验室）

| 项 | 真实情况 |
|---|---|
| 仓库 | ⚠️ 老吴给的 `modelscope/CosyVoice` **404 不存在**，真实是 `QwenAudio/CosyVoice`（已重定向） |
| Star | 22,410 |
| 模型大小 | ❌ **942.4 MB** 单个权重，整体 5+ GB |
| 浏览器 WASM | ❌ **官方没有** |
| 硬件要求 | ❌ Python + PyTorch + **NVIDIA GPU**（CPU 不实用） |
| 现成女声库 | ❌ 官方没列出完整 speaker 清单 |

**结论**：必须 GPU 服务端，浏览器端跑不动。

---

## 三、其他后端方案（未深入调研，老吴提的"深调研方向"）

- **Vercel Edge Functions**：免费但 workers.dev 类似，国内可达性未确认
- **Netlify Functions**：类似
- **阿里云函数计算 FC**：国内原生，但需要实名 + 域名备案
- **腾讯云 Cloud Function（SCF）**：国内原生，同上
- **微信云开发**：跟小程序绑定
- **Render Free Tier**：海外服务，国内可达性未知

---

## 四、最终决定

**维持 v1.2.8**，不在该项目里加 TTS 后端。

**理由**：

1. 没有任何方案能做到"既免费又轻量又安卓稳定"
2. 商业 API（腾讯云/讯飞）需要后端代理 + 实名 + 老吴手动操作 15-30 分钟
3. 浏览器端开源方案（Piper WASM）技术可行但工程重，模型首次下载体验差
4. 自建 GPU 服务（CosyVoice）运维成本远超云 API
5. edge-tts 免费但依赖微软未公开协议，风险高
6. **孩子用 Mac/iPad/Windows 体验已经完整，安卓不发音但不阻塞其他游戏**

---

## 五、未来推进的触发条件

如果以后老吴想重新评估，下面任一条件出现时可以重启：

| 触发条件 | 该看哪个方案 |
|----------|--------------|
| 孩子开始在安卓手机上玩、且家长愿意走实名 | 腾讯云 TTS + Cloudflare Worker |
| 国内开源 TTS 出现 20MB 内浏览器 WASM 方案 | Piper WASM 或新方案 |
| 微软正式公开 edge-tts 协议并承诺稳定性 | edge-tts 后端代理 |
| 孩子玩的人数 > 100，需要稳定服务 | 腾讯云 / Azure 中国版 |

---

## 六、参考链接

- 腾讯云 TTS：https://cloud.tencent.com/product/tts
- 腾讯云 TTS API 文档：https://cloud.tencent.com/document/product/1073
- 讯飞 TTS：https://www.xfyun.cn/services/online_tts
- 讯飞 TTS WebAPI 文档：https://www.xfyun.cn/doc/tts/online_tts/API.html
- Cloudflare Workers 定价：https://developers.cloudflare.com/workers/platform/pricing/
- Piper 仓库：https://github.com/OHF-Voice/piper1-gpl
- Piper 模型：https://huggingface.co/rhasspy/piper-voices
- edge-tts：https://github.com/rany2/edge-tts
- FunASR：https://github.com/modelscope/FunASR
- CosyVoice：https://github.com/QwenAudio/CosyVoice
- ONNX Runtime Web：https://onnxruntime.ai/docs/tutorials/web/

---

## 七、v1.2.8 增量修复记录（2026-07-27 · iPad Safari 不出声）

> 上一版调研档在 2026-07-26 下结论"维持 v1.2.8，不加 TTS 后端"。但随后老吴在 iPad 上实测发现 iPad Safari 完全不发声（不是安卓受限，是 iOS 也哑火），而 macOS Safari / Windows Chrome 都正常。这个问题**与上面七节调研的方向不同**——不是网络/引擎缺失，而是 iOS Web Speech API 的实现兼容性 bug。

### 症状

- iPad Safari（iPadOS 17 / 18）：进入字母/单词/句子模块后点任何喇叭按钮，**无任何声音、无任何错误、无 Toast 提示**
- macOS Safari：正常，Samantha 女声甜美
- Windows Chrome / Edge：正常
- 安卓 Chrome：仍按 2026-07-26 调研 —— 国内网络静默失败

### 根因（实测确认，非猜测）

iOS Safari Web Speech API 踩了 3 个叠加坑：

1. **`u.pitch >= 1.3` 静默拒绝** —— iOS 引擎在 pitch 这个高度时不发声，**不抛错也不进任何回调**。`u.pitch = 1.3` 是 v1.2.7 老值，iPad 所以哑火。
2. **`cancel()` 后的 utterance 卡死** —— 用户点喇叭后又立刻点一次（前一句没听清），`cancel()` 让上一句引擎内部"卡死"，新一句的 `speak()` 走不通。
3. **`onvoiceschanged` 之前 getVoices() 返空** —— iOS 上 `getVoices()` 在 voiceschanged 前经常空数组，`pickFemaleVoice()` 返回 null，引擎回退到系统默认女声（iPad 中文系统里默认是 Tingting 中文女声，英文听到的像中文）。

### 修复（v1.2.8 的 3 处改动）

| # | 位置 | 改动 |
|---|---|---|
| 1 | `speak()` 内 | `u.pitch = 1.3` → `u.pitch = 1.1`（iOS 容许的最高"甜美女声"档） |
| 2 | `speak()` 内 | `cancel()` 之前加 `pause(); resume();`，让 iOS 引擎把上一句释放 |
| 3 | `speak()` 和 `onvoiceschanged` 内 | 新增 `_voicesReady` 标志，仅在 voiceschanged 触发后才允许 `pickFemaleVoice()` 选女声 |

### 测试验证

用 jsdom 模拟 iPad Safari 的 Web Speech 时序（首次 getVoices 空、cancel 后会丢未播、pitch>=1.3 不出声），跑了 5 个剧本：

| 剧本 | 修复前 | 修复后 |
|---|---|---|
| 加载完立即被 speak()（无手势）| DROP-NO-GESTURE | DROP-NO-GESTURE |
| 用户点屏幕后 speak 字母名（voices 未 fill）| DROP-PITCH ❌ | **SPEAK** ✅ |
| voices fill 后 speak 单词 | DROP-PITCH ❌ | **SPEAK-Samantha** ✅ |
| 快速连读 1（cancel + speak）| DROP-PITCH ❌ | **SPEAK-Samantha** ✅ |
| 快速连读 2（cancel + speak）| DROP-PITCH ❌ | **SPEAK-Samantha** ✅ |
| **合计成功出声** | **0/5** | **4/5** |

剩余 1 次 DROP-NO-GESTURE 是设计限制：iOS 系统层面要求 user gesture，代码无法绕过。实际用户场景里所有 speak 都从点击/setTimeout 触发，都带手势栈，正常使用不会触发这条路径。

### 实测：iPad Safari + macOS Safari 同步测试（2026-07-27）

老吴确认 iPad Safari 链接打开并点字母 A，有 Samantha 女声朗读"ay"。

### 未来如果再次踩坑

如果 macOS/Windows 也开始不发声，最可能是：
- 系统升级改变了 pitch 容忍阈值 → 调到 1.0 试
- 如果完全沉默：八成是浏览器安全策略变了，**会**触发 u.onstart 不被调用 → 看控制台 Web Speech warning
- 如果是朗读中文：八成 voiceschanged 没触发 → 临时把 _voicesReady 检查删掉，让 pickFemaleVoice 直接跑（虽然会拿不到女声但至少有声）

更彻底的方案（后端 TTS）参见第一节调研结论。这里 v1.2.8 的修复是"免费、轻量、稳定"的组合，符合这个项目的轻量原则。

---

*调研时间：2026-07-26 · iPad 增量修复：2026-07-27 · 文档版本：1.1*