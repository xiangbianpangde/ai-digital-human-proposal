# AI 视频生成技术调研

> 归属：`00-资料库/02-AI视频/` ｜ 遵循 `_规范.md` 证据协议
> 建立日期：2026-09-29 ｜ **全库统一查询日期：2026-09-29**（凡涉及定价/star/许可证，均以此日为准）
> 适用项目：AI 口播数字人 — 产研实习合作提案

---

## 本节速览

1. **[事实] Google 官方最新视频模型是 Veo 3.1，不存在官方"Veo 4"。** Google DeepMind 官方模型页标题即为「Veo 3.1」（`deepmind.google/models/veo/`，页面 og:title 实测）。搜索中大量"Veo 4 于 2026 年 4 月发布"的说法全部来自第三方 SEO 站点，**未定位到 Google 任何官方发布** → 不得写入 PPT。
2. **[事实] 闭源旗舰的真实竞争格局在 2026 年已经变成"按秒计价的音视频一体模型"。** Veo 3.1 三个档位 $0.05–$0.40/秒、Gemini Omni Flash ≈$0.10/秒、字节 Seedance 2.0 原生音视频、可灵 v3 turbo 恒定出音轨。价格区间已从"几十美元一分钟"压缩到**约 $5–25 / 分钟成品素材**。
3. **[事实] 提案 brief 里的两个版本号已过时。** 快手最新为 **Kling v3**（非 2.x），字节最新为 **Seedance 2.0**（2026 年 2 月中国区发布），Runway 最新为 **Gen-4.5**（非 Gen-4）。Lightricks 最新为 **LTX-2.x**（非 LTX-Video 0.9）。
4. **[事实] 开源模型的商用许可呈两极分化，且这是提案最硬的成本论据。** Wan 系列为 **Apache-2.0**（真可商用）；**HunyuanVideo 为腾讯社区许可，排除欧盟/英国/韩国三大法域**；**LTX-2.x 为社区许可，年收入 ≥ $10,000,000 的实体商用必须另签付费许可**。三者法律后果完全不同。
5. **[事实] 画质不是当前瓶颈，可控性才是。** OpenAI 官方文档主动承认 Sora 2 在"精确的基于时间的镜头运动编排（precise time-based event sequencing such as camera movement）"上存在困难；微软文档同时列出 Sora 2 只能"最多用两张图作为输入（在两者之间插值）"。这两条是**厂商自证的缺陷**，比任何第三方评测都更有说服力。
6. **[事实] "storyboard → shot" 已进入商用 API 的官方参数层。** 阿里云百炼官方文档为 Kling v3 直接提供「文生视频（智能分镜）」接口，以及参考生视频、视频编辑接口——工作流不再只是从业者的土法。
7. **[事实] 中国合规有一条容易被漏掉的硬要求：AI 视频必须打标。** 《人工智能生成合成内容标识办法》（国信办通字〔2025〕2 号，2025-09-01 施行）要求视频在**起始画面和播放周边**加显式标识，并在**文件元数据**中加隐式标识，配套强制性国标同步实施。提案若无标识合规设计，是可被当场否掉的。
8. **[推断] 本项目（口播数字人 + AI 视频）在"自部署 vs 调 API"上应当选 API 为主、自部署为辅。** 理由：可控性要求（首尾帧、角色一致、口型对齐）当前最强的一批能力都在闭源侧（Kling-MotionControl、Kling-Omni、VACE 等）；而开源侧唯一真正宽松商用的是 Wan（Apache-2.0），但它没有口型/说话人驱动管线。且自部署要额外背 GPU 采购、生成 5 秒 480P 需约 4 分钟（RTX 4090，官方 README 数据）这类**吞吐劣势**，对需要日更的房产内容不成立。详见文末成本推演。

---

## 1. 闭源旗舰视频模型现状

### [E-A-01] Google Veo 3.1 —— 官方当前旗舰，无 Veo 4
- **事实**：Google DeepMind 官方 Veo 模型页的 `og:title` 与页面标题均为「Veo 3.1」，页面描述为「Introducing our leading video generation model Veo 3.1, and new capabilities for Veo 2.」；截至查询日，Google 官方渠道无任何名为 "Veo 4" 的模型、产品页、模型卡或定价条目。
- **来源类型**：公司官网/新闻稿（模型页）
- **证据等级**：A
- **来源**：
  - URL: https://deepmind.google/models/veo/
  - 发布/更新日期: 页面未标注发布日期（Veo 3.1 官方首发为 2025-10，本文按未标注处理）
- **支撑细节**：多个第三方站点（veo3ai.io、ourdream.video、evolink.ai 等）宣称"Veo 4 已于 2026 年 4 月发布"，其自述互相矛盾（如 aireiter.com 称"Google I/O 2026 未发布 Veo 4"）。**官方页面不支持这些说法。**
- **我的推断**：Veo 3.1 仍是 Google 唯一在售的视频生成主线模型；Gemini Omni Flash 是其新增的低价音视频线（见 E-A-03），而非 Veo 4 的继任命名。
- **可用于 PPT**：✅（作为"当前旗舰版本号"的唯一权威依据）

### [E-A-02] Veo 3.1 官方 API 定价（三档，按秒计费）
- **事实**：Google 官方 Gemini Developer API 定价页列出 Veo 3.1 三个变体的每秒价格（Paid Tier，USD/秒）：**Standard** $0.40（720p 与 1080p）、$0.60（4K）；**Fast** $0.10（720p）、$0.12（1080p）、$0.30（4K）；**Lite** $0.05（720p）、$0.08（1080p，不支持 4K）。免费层不提供 Veo。页面另注明：若因音频处理问题导致生成失败则不计费，仅成功生成才计费。
- **来源类型**：官方技术文档 / 官方定价页
- **证据等级**：A
- **来源**：
  - URL: https://ai.google.dev/gemini-api/docs/pricing （章节 `id="veo-3.1"`）
  - 查询日期: 2026-09-29（页面标注 Last updated 2026-09-24 UTC）
- **支撑细节**：模型 ID 为 `veo-3.1-generate-preview`、`veo-3.1-fast-generate-preview`、`veo-3.1-lite-generate-preview`。
- **我的推断**：三档价差达 8 倍，说明 Google 在用价格而非能力分层；对"批量生成、抽卡重试"的业务模式，Fast 档（$0.10/s）比 Standard 档（$0.40/s）合理得多。
- **可用于 PPT**：✅（价格必须标注"查询日期 2026-09-29"）

### [E-A-03] Gemini Omni Flash —— Google 新一代音视频一体模型
- **事实**：Gemini API 定价页列出 `gemini-omni-1.1-flash`（Gemini Omni Flash），官方描述为「Our next-generation video generation and editing model」，已在 Gemini API 付费层**正式可用（generally available）**。定价：输入 $1.50 / 100 万 tokens（文本/图像/视频/音频统一价）；输出文本 $9.00、视频 $17.50 / 100 万 tokens。官方按 **5,792 tokens / 秒（720p）** 折算，**等效约 $0.10 / 秒**。预览版 `gemini-omni-flash-preview` 同价。
- **来源类型**：官方定价页 / 官方技术文档
- **证据等级**：A
- **来源**：
  - URL: https://ai.google.dev/gemini-api/docs/pricing （章节 `id="gemini-omni-1.1-flash"`）
  - 查询日期: 2026-09-29
- **我的推断**：这是 Google 生态内性价比最高的视频档位（约 $0.10/s，与 Veo 3.1 Fast 同价但为音视频一体），值得作为与 Veo 并列的备选写入方案比选表。
- **可用于 PPT**：✅

### [E-A-04] OpenAI Sora 2 —— 官方能力与技术限制（第三方转载的官方文档）
- **事实**：Sora 2 支持 text→video、image→video、video→video，输出**带音频**，并支持 **Remix**（对已有视频做定向修改而非重新生成）。分辨率白名单：480x480、480x854、854x480、720x720、720x1280、1280x720、1080x1080、1080x1920、1920x1080；时长 **1–20 秒**。并发限制为同时 2 个生成任务。输入限制：**最多使用两张图**（模型在两者间插值）、**最多使用 5 秒视频**作为输入。1080p 不支持多变体；720p 最多 2 个变体。
- **来源类型**：官方技术文档（Microsoft Foundry / Azure OpenAI 官方文档，明确标注 "Adapts OpenAI's latest Sora 2 API using v1 API"）
- **证据等级**：A
- **来源**：
  - URL: https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/video-generation
  - 发布/更新日期: `ms.date` 2026-03-18；`updated_at` 2026-06-05
- **我的推断**：**"最多两张输入图"对本项目是硬约束**——若要做"多机位/多角色房产讲解"需要更多参考图，Sora 2 单次调用无法满足，必须改用支持多参考图的模型（Kling 参考生视频、Seedance 2.0 多模态参考）。
- **可用于 PPT**：✅

### [E-A-05] Sora 2 官方自述的技术缺陷（对"可控性比画质更关键"的直接证据）
- **事实**：微软官方文档明确列出 Sora 2 的内容质量限制原文：「Sora 2 might have difficulty with **complex physics, causal relationships**（例如饼干上的咬痕）、**spatial reasoning**（例如分清左右）、and **precise time-based event sequencing such as camera movement**.」另注：Sora 2 阻止生成所有 IP 与写实人像内容（"Sora 2 blocks all IP and photorealistic content"）。
- **来源类型**：官方技术文档
- **证据等级**：A
- **来源**：
  - URL: https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/video-generation
  - 发布/更新日期: 2026-03-18
- **支撑细节**：同页 Best Practices 另注「Write text prompts in English or other **Latin script** languages for the best video generation performance.」——对中文房产口播脚本的提示词工程是直接风险。
- **我的推断**：**镜头运动控制当前仍是 Sora 2 的公认短板**，这与 Veo/Kling 主推运镜控制形成对比。对需要"推/摇/移/跟"编排的讲解类视频，Sora 2 不是首选。
- **可用于 PPT**：✅（这是全库最有价值的一条：厂商自证，比第三方评测更硬）

### [E-B-06] Sora 2 API 单价 —— 未取到官方原始出处
- **事实**：OpenAI 官方定价页与 `openai.com/index/sora-2/` 在本次查询中均被 Cloudflare 人机校验拦截，**未能取得官方单价数字**。微软文档仅给出计费口径为「按秒计费（Per second billing）」并链接到 Azure 定价页，本次亦未渲染出数字。
- **来源类型**：官方定价页（未取到）
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: https://platform.openai.com/docs/pricing （查询 2026-09-29，被拦截）
  - 备选: https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/
- **我的推断**：无。**禁止在 PPT 中写入任何 Sora 单价数字**，只能写"按秒计费，官方单价待核验"。
- **可用于 PPT**：❌（补齐后才能用）

### [E-A-07] Runway Gen-4.5 与 credits 计价（官方定价页，可反推单价）
- **事实**：Runway 官方定价页给出订阅与 credits 换算：Standard $12/月（年付）625 credits = **52 秒 Gen-4.5**；Pro $28/月 2250 credits = **187 秒 Gen-4.5**；Max $76/月 9500 credits = **791 秒 Gen-4.5**。「Compare models」表列出 **Gen-4.5 = 60 credits / 5 秒**，Aleph 2.0 = 140 credits / 5 秒，Seedance 2.0 Pro 1080p = 160 credits / 4 秒。Pro 层起提供音乐/音效生成、无水印、4K 超分。
- **来源类型**：官方定价页
- **证据等级**：A
- **来源**：
  - URL: https://runwayml.com/pricing
  - 查询日期: 2026-09-29
- **支撑细节**：由 60 credits/5s 换算 = **12 credits/秒**；Max 层 9500 credits ÷ 12 ≈ 791s，与官方自述一致（内部自洽，可交叉验证）。
- **我的推断**：Gen-4.5 折合约 **$0.15/秒**（Max 层 9500 credits / $76 ≈ $0.0096/credit × 12）。**Runway 的真正价值不是单价，而是 credits 可滚存 + 无水印 + 4K 超分**，适合成片阶段的收尾工序。
- **可用于 PPT**：✅（单价须写明"由 credits 反推"，标注为推断）

### [E-B-08] Runway 已聚合第三方旗舰模型（Kling 3.0 / Seedance 2.5）
- **事实**：Runway 官方定价页的套餐包含列表中，Pro 及以上档位明确列出 **Seedance 2.5、Gen-4.5、Kling 3.0、Nano Banana Pro**；对比表另列 **Aleph 2.0** 与 **Seedance 2.0 Pro 1080p**。即 Runway 自身旗舰之外，同时代售第三方模型。
- **来源类型**：官方定价页
- **证据等级**：A（作为"Runway 产品线构成"的事实）
- **来源**：
  - URL: https://runwayml.com/pricing
  - 查询日期: 2026-09-29
- **我的推断**：**独立第三方同时在售 Kling 3.0 与 Seedance 2.5，反向证明这两个版本号是当前在售主力**（可作为 E-A-09、E-A-10 的交叉印证）。
- **可用于 PPT**：✅（用于论证"版本号已迭代到 v3 / 2.5"）

### [E-A-09] 快手可灵 Kling v3 —— 版本、能力与计费口径
- **事实**：阿里云百炼（Model Studio）官方文档中，可灵在售模型为 `kling/kling-v3-video-generation`、`kling/kling-v3-turbo-video-generation`、`kling/kling-v3-omni-video-generation`。**时长 3–15 秒（默认 5 秒），按秒计费**；`audio` 为布尔参数（开启则生成匹配音景的配音/音效），但 **`kling-v3-turbo` 固定生成音画同出视频，该字段恒为生成、不生效**；传入视频时 audio 只能为 false。`watermark` 可选（文案固定为"可灵 AI"）。官方提供 **文生视频（智能分镜）**、参考生视频（feature+first_frame）、视频编辑等接口。调用为异步任务（DashScope `VideoSynthesis.async_call`），华北2（北京）地域。
- **来源类型**：官方技术文档（阿里云官方帮助中心，代售可灵模型）
- **证据等级**：A
- **来源**：
  - URL: https://help.aliyun.com/zh/model-studio/kling-video-generation-api-reference
  - 发布/更新日期: 2026-09-04（搜索引擎收录的该页更新日期）
- **支撑细节**：文档原文「`duration` **直接影响费用，按秒计费**，时间越长费用越高，请前往百炼控制台查看价格」——即单价在控制台，文档不含数字。
- **我的推断**：可灵 v3 已把"音画同出"设为 turbo 档默认，说明**口播数字人所需的原生音轨，2026 年在旗舰模型上已是标配而非卖点**；选型时该维度可降权，权重应转向角色一致性与运镜可控性。
- **可用于 PPT**：✅

### [E-B-10] 可灵官方定价页 —— JS 动态渲染，本次未取到可核验单价
- **事实**：`klingai.com/dev/pricing` 页面 HTML 仅含 SEO 骨架（提取正文仅 213 字符），价格表由 JavaScript 动态加载；抓取到的仅为站点配置与商务表单定义（含"企业版创作成果归属企业、绝不用企业数据训练模型"等**企业条款表述**），**无任何单价数字**。
- **来源类型**：官方定价页（未取到）
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: https://klingai.com/dev/pricing （查询 2026-09-29）
- **支撑细节**：同一配置块中出现可用的**企业侧事实**：Kling AI 企业会员"创作成果归属企业""绝不挪用企业数据训练模型""支持原生 4K 直出""支持多主体参考创作""高并发"。
- **我的推断**：企业版"数据不用于训练 + 成果归属企业"是**商用合作谈判中最值钱的一条**，可在提案中作为需向可灵方确认的条款清单，但不可写成已获得的承诺。
- **可用于 PPT**：⚠️ 仅可作为"待确认商务条款"，不可写单价

### [E-A-11] 字节 Seedance 2.0 —— 原生多模态音视频模型
- **事实**：Seedance 2.0 技术报告称其为"原生多模态音视频生成模型"，**2026 年 2 月初在中国区正式发布**，采用统一的高效大规模音视频联合生成架构，支持 **文本、图像、音频、视频四种输入模态**。官方声明：支持 **4–15 秒**的音视频内容直接生成，**原生输出 480p 与 720p**；作为多模态参考输入时，当前开放平台支持**最多 3 段视频、9 张图、3 段音频**；另提供 Seedance 2.0 Fast 加速版本。
- **来源类型**：论文原文（厂商自投 arXiv）+ 公司官网
- **证据等级**：A（作为"官方公布的规格"）
- **来源**：
  - URL: https://arxiv.org/abs/2604.14148 （Seedance 2.0: Advancing Video Generation for World Complexity）
  - 官方页: https://seed.bytedance.com/seedance2_0
  - 发布/更新日期: arXiv published 2026-04-15（论文自述发布于 2026 年 2 月初）
- **支撑细节**：9 张参考图的上限**显著高于 Sora 2 的 2 张**，且覆盖视频/音频参考，是多模态参考型工作流的关键差异点。
- **我的推断**：**就"参考素材数量"这一项，Seedance 2.0 是当前旗舰中对多参考图最友好的**，适合房产讲解中"户型图 + 样板间实拍 + 小区环境"的多源素材约束生成。
- **可用于 PPT**：✅（"9 图 / 3 视频 / 3 音频"是可写进 PPT 的硬数字）

### [E-A-12] Seedance 2.0 的"领先水平"属厂商宣称，无独立复现
- **事实**：论文原文表述为「In both **expert evaluations and public user tests**, the model has demonstrated performance **on par with the leading levels** in the field.」——此为字节自评，未公布评测集细节、参数量、对照组版本或可复现协议。
- **来源类型**：论文原文（厂商自投）
- **证据等级**：C ⚠️ 未核验（作为能力主张）
- **来源**：
  - URL: https://arxiv.org/abs/2604.14148
  - 发布/更新日期: 2026-04-15
- **我的推断**：按 `_规范.md` 第 6 条，此类数字在 PPT 附录须标注"厂商宣称，无独立第三方复现，测试条件未公开"。
- **可用于 PPT**：⚠️ 仅可出现在附录，且必须标注厂商宣称

### [E-C-13] Midjourney Video / Luma Dream Machine / Pika —— 未取到可核验规格与定价
- **事实**：本次调研**未定位到**这三家官方定价页与模型规格的可抓取原文（其定价页同属 JS 动态渲染，与可灵同类问题）。不写入任何未经核验的数字。
- **来源类型**：无
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: 待补（Midjourney `docs.midjourney.com`、Luma `lumalabs.ai/dream-machine`、Pika `pika.art`）
  - 发布/更新日期: 未标注
- **我的推断**：无。
- **可用于 PPT**：❌（建议在 PPT 中直接不列这三家，或仅作"其他工具"一笔带过）

### [E-C-14] Sora 产品线状态 —— 存在互相矛盾的说法，未采信
- **事实**：检索中出现两个互斥说法：OpenAI 官方 Sora 2 发布页的搜索摘要显示「The **Sora product is no longer available**」；另有第三方站点（terabox，2026-09-24）称"2026 年应用关停"。因 `openai.com` 被 Cloudflare 拦截，**未能打开原文核实**，两个说法均无法确认。
- **来源类型**：搜索摘要 / 自媒体
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: https://openai.com/index/sora-2/ （查询 2026-09-29，被拦截）
  - 旁证: https://insights.terabox.com/hub/what-happened-to-openai-sora-and-how-to-use-sora-2-today
- **我的推断**：**这是一条高风险假设**——若 Sora 消费端确已关停，则 Sora 仅剩 API 形态可用，方案选型中应把 Sora 降级为"备选"而非"首选"。**必须在提案前用可访问网络核实。**
- **可用于 PPT**：❌（核实前不得提及产品状态）

---

## 2. 开源 / 可自部署视频模型（许可证是核心）

### [E-A-15] Wan2.2（阿里巴巴通义万相）—— Apache-2.0，唯一无商用障碍的旗舰开源
- **事实**：Wan-Video/Wan2.2 仓库许可证为 **Apache License 2.0**（`LICENSE.txt` 首行为 "Apache License / Version 2.0, January 2004"），GitHub API 报告 `license.spdx_id = Apache-2.0`。开源模型清单：**T2V-A14B**（MoE，支持 480P & 720P）、**I2V-A14B**（MoE，480P & 720P）、**TI2V-5B**（高压缩 VAE 16×16×4，T2V+I2V，**支持 720P@24FPS，可跑 RTX 4090**）、**S2V-14B**（语音驱动，480P & 720P）、**Animate-14B**（角色动画与替换）。已集成 Diffusers 与 DiffSynth-Studio（逐层 offload、FP8 量化、序列并行、LoRA 训练）。相对 Wan2.1 训练数据 +65.6% 图像、+83.2% 视频。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Wan-Video/Wan2.2
  - 查询日期: 2026-09-29
  - 关键数据（star / 最后 push）：**17,663 stars**；最后 push 2026-09-21；仓库创建 2025-07-28
- **支撑细节**：Apache-2.0 允许商用、修改、再分发，仅需保留版权声明与变更说明，**无营收门槛、无地域排除**。
- **我的推断**：**这是本项目自部署路线唯一"法律上干净"的选项。** 若合作方要求资产不出域，Wan2.2-TI2V-5B（可跑 4090）是唯一现实的起点。
- **可用于 PPT**：✅（许可证 + star + 参数量均可写）

### [E-A-16] Wan2.1 消费级显卡门槛（自部署成本论据的硬数据）
- **事实**：Wan2.1 官方 README 原文：「The **T2V-1.3B** model requires only **8.19 GB VRAM**, making it compatible with almost all consumer-grade GPUs. It can generate a **5-second 480P video on an RTX 4090 in about 4 minutes**（without optimization techniques like quantization）」。Wan2.1 仓库许可证同为 Apache-2.0。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Wan-Video/Wan2.1
  - 查询日期: 2026-09-29
  - star / push：**17,063 stars**；最后 push 2026-03-05
- **我的推断**：**单卡 4090 约 4 分钟产出 5 秒 480P 素材 → 约 48 秒素材/分钟机时 → 生成 1 分钟 720P 成片素材，单卡约需 25–60 分钟机时**（按 Wan2.2 720P 分辨率更高再乘 2–4 倍估算）。**这对日更数十条房产视频的吞吐是致命的**，是"自部署不划算"的最硬论据。
- **可用于 PPT**：✅（必须标注为"官方 README 数据 + 我的换算推断"）

### [E-A-17] HunyuanVideo-1.5（腾讯混元）—— 8.3B，但许可证排除三大法域
- **事实**：HunyuanVideo-1.5 官方 README 称「delivers top-tier quality with only **8.3B parameters** … runs smoothly on **consumer-grade GPUs**」。仓库 LICENSE 文件为 **TENCENT HUNYUAN COMMUNITY LICENSE AGREEMENT**（License date: **November 21, 2025**），开宗明义：**「THIS LICENSE AGREEMENT DOES NOT APPLY IN THE EUROPEAN UNION, UNITED KINGDOM AND SOUTH KOREA AND IS EXPRESSLY LIMITED TO THE TERRITORY」**，且 Territory 定义为「the worldwide territory, **excluding** the territory of the European Union, United Kingdom and South Korea」。GitHub API 对该仓库返回 `spdx_id = NOASSERTION`（即非标准 SPDX 许可证）。
- **来源类型**：官方仓库 + 许可证原文
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5
  - 许可证原文: https://raw.githubusercontent.com/Tencent-Hunyuan/HunyuanVideo-1.5/main/LICENSE
  - 查询日期: 2026-09-29
  - star / push：**4,561 stars**；最后 push 2026-04-10
- **支撑细节**：已提供 ComfyUI 集成与 LightX2V 加速支持；技术报告 arXiv:2511.18870（发布 2025-11-24）。
- **我的推断**：8.3B + 消费级可跑是显著进步，但**腾讯社区许可是"有条件开源"**——境内商用可行，跨境部署/多区域服务需法务单独确认。**不能与 Wan 的 Apache-2.0 并列为"可自由商用"。**
- **可用于 PPT**：✅（许可证差异必须写清，这是选型分水岭）

### [E-A-18] HunyuanVideo（初版）许可证为 NOASSERTION，不可按开源处理
- **事实**：Tencent-Hunyuan/HunyuanVideo 仓库 **根目录无 `LICENSE` 文件**（`raw.githubusercontent.com/.../main/LICENSE` 返回 404），GitHub API 返回 `spdx_id = NOASSERTION`。其使用条款指向腾讯混元通用社区许可（发布日 2025-11-21）。
- **来源类型**：官方仓库（许可证缺失为事实）
- **证据等级**：A（对"该仓库无可识别开源许可证"这一事实）
- **来源**：
  - URL: https://github.com/Tencent-Hunyuan/HunyuanVideo
  - 查询日期: 2026-09-29
  - star / push：**12,574 stars**；最后 push 2026-06-29
- **我的推断**：若需使用混元系模型，**必须选 HunyuanVideo-1.5 而非初版**，并按腾讯社区许可逐条过法务。
- **可用于 PPT**：✅（作为"许可证核对表"的一行）

### [E-A-19] LTX-2 / LTX-2.x（Lightricks）—— 社区许可，$10M 营收门槛
- **事实**：Lightricks/LTX-2 为 "Official Python inference and LoRA trainer package for the LTX-2 audio–video generative model"，仓库含三份许可：`LICENSE`、`LICENSE-2`（**LTX-2 Community License Agreement, dated January 5, 2026**，适用于 2026-01-05 起的 LTX-2.x 版本至 2026-08-11）、`LICENSE-2_x`（**LTX-2.x Community License Agreement, dated August 11, 2026**，适用于 2026-08-11 起的全部 LTX-2.5 及后续版本）。GitHub API 返回 `spdx_id = NOASSERTION`（非标准许可）。**关键条款（LICENSE-2_x 第 2.1 条原文）**：「Entities with **annual revenues of at least $10,000,000**（the "Commercial Entities"）are required to **obtain a paid license** for any use … of LTX-2.x and Derivatives」，联系邮箱 ltxv-licensing@lightricks.com；未按协议使用的「strictly prohibited and shall be deemed a **material breach**」，且须在书面要求后 30 天内补交按商业许可费率计算的许可费。第 2.2 条进一步澄清：营收性活动、与终端用户直接交互、为商业用途训练/微调/蒸馏，**均不属"非商业目的"**。
- **来源类型**：官方仓库 + 许可证原文
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Lightricks/LTX-2
  - 许可证原文: https://raw.githubusercontent.com/Lightricks/LTX-2/main/LICENSE-2_x
  - 查询日期: 2026-09-29
  - star / push：**9,545 stars**；仓库创建 2026-01-03；最后 push 2026-08-26
- **我的推断**：**实习项目/初创公司体量远低于 $10M 营收，落在免费额度内**；但若合作方是有营收的成熟企业（尤其年营收过千万或计划融资后规模化），**LTX-2.x 会在某一天突然变成必须付费**。这是一个应在提案里点明、而非默认忽略的条款。
- **可用于 PPT**：✅（"开源≠免费商用"的最佳教材；阈值数字须原样引用）

### [E-A-20] LTX-Video（初版）为 Apache-2.0，但已被 LTX-2 取代
- **事实**：Lightricks/LTX-Video 仓库许可证为 **Apache-2.0**（`spdx_id = Apache-2.0`），**10,994 stars**，最后 push 2026-01-05。其 README 明确：「[LTX-2](https://github.com/Lightricks/LTX-2) … **is now the primary home for LTX development**」，LTX-2 具备同步音视频生成、**multiple keyframe support**、**IC-LoRA control models**、latent upsampler，并**已内置 ComfyUI core**。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Lightricks/LTX-Video
  - 查询日期: 2026-09-29
- **我的推断**：想自部署 + 要 Apache-2.0 + 要音频，只能回到**旧的 LTX-Video**（无音视频一体），或接受 LTX-2.x 的社区许可条件。**这是"开源侧能力"与"开源侧许可宽松度"之间的真实取舍。**
- **可用于 PPT**：✅

### [E-A-21] Mochi 1（Genmo）—— Apache-2.0，但已停更
- **事实**：genmoai/mochi（仓库已从 `genmoai/models` 重定向至此）README 原文：「We're releasing the model under a **permissive Apache 2.0 license**」；支持 LoRA 微调（2024-11-26 加入）与 ComfyUI 原生支持（2024-11-05）。GitHub API：`Apache-2.0`，**3,733 stars**，最后 push **2025-11-14**。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/genmoai/mochi
  - 查询日期: 2026-09-29
- **我的推断**：许可最宽松，但 2025-11 后基本停更，**不建议作为 2026 年新项目的技术起点**（可作为对照基线）。
- **可用于 PPT**：⚠️（列入备选表末位）

### [E-A-22] CogVideoX（智谱）—— Apache-2.0，star 量最高的开源视频模型之一
- **事实**：仓库已迁移至 `zai-org/CogVideo`（原 `THUDM/CogVideo` 返回 301 Moved Permanently）。GitHub API：**13,048 stars**，许可证 **Apache-2.0**，最后 push **2025-11-04**。README 记录：`CogVideoX1.5-5B` 支持 **10 秒**视频且分辨率更高，`CogVideoX1.5-5B-I2V` 支持任意分辨率生成；提供 `cogvideox-factory` 单张 4090 微调框架；CogKit 作为 CogView4/CogVideoX 系列的微调与推理框架。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/zai-org/CogVideo
  - 查询日期: 2026-09-29
- **我的推断**：star 数仅次于 Wan2.2，说明**社区关注度高**，但同样 2025-11 后停更；10 秒生成长度在 2026 年已落后于闭源旗舰（可灵 15 秒、Sora 20 秒）。
- **可用于 PPT**：✅（列入备选表，标注"更新停滞"）

### [E-A-23] Step-Video-T2V（阶跃星辰）—— MIT，但体量与更新频率最小
- **事实**：stepfun-ai/Step-Video-T2V 许可证 **MIT**（GitHub API `spdx_id = MIT`），**3,185 stars**，最后 push **2025-03-17**。技术报告 arXiv:2502.10248「Step-Video-T2V Technical Report」（发布 2025-02-14）。另有 `Step-Video-TI2V`（图生视频，374 stars，MIT）。
- **来源类型**：官方仓库 + 论文原文
- **证据等级**：A
- **来源**：
  - URL: https://github.com/stepfun-ai/Step-Video-T2V
  - URL: https://arxiv.org/abs/2502.10248
  - 查询日期: 2026-09-29
- **我的推断**：MIT 许可最宽松，但 2025-03 后停更、体量最小。**列入清单仅作生态完整性，不建议投入。**
- **可用于 PPT**：⚠️（附录备选表）

### [E-A-24] 开源模型许可证总览（横向对比，提案可直接引用）
- **事实**：本次查询逐一核实的许可证结果如下（全部以 2026-09-29 GitHub API + 仓库 LICENSE 原文为准）：

  | 模型 | 许可证 | 是否可商用 | 关键限制 | Stars |
  |---|---|---|---|---|
  | Wan2.2 | Apache-2.0 | ✅ 可 | 需保留声明 | 17,663 |
  | Wan2.1 | Apache-2.0 | ✅ 可 | 需保留声明 | 17,063 |
  | CogVideoX (zai-org) | Apache-2.0 | ✅ 可 | 需保留声明 | 13,048 |
  | Mochi 1 | Apache-2.0 | ✅ 可 | 需保留声明 | 3,733 |
  | Step-Video-T2V | MIT | ✅ 可 | 保留声明 | 3,185 |
  | HunyuanVideo-1.5 | 腾讯混元社区许可 | ⚠️ 有条件 | **排除欧盟/英国/韩国法域** | 4,561 |
  | HunyuanVideo（初版） | 无 LICENSE 文件（NOASSERTION） | ❌ 不可推定 | 须逐条咨询 | 12,574 |
  | LTX-2.x | LTX 社区许可 | ⚠️ 有条件 | **年营收 ≥$10M 须付费许可** | 9,545 |
  | LTX-Video（初版） | Apache-2.0 | ✅ 可 | — | 10,994 |

- **来源类型**：官方仓库 / GitHub API
- **证据等级**：A
- **来源**：
  - URL: https://api.github.com/repos/{owner}/{repo} （逐仓库查询）
  - 查询日期: 2026-09-29
- **我的推断**：**8 个模型中只有 4 个是"真正可商用"的无条件开源，2 个有实质限制，1 个无法推定，1 个已停更。**"开源"与"能商用"在本行业是两件事。
- **可用于 PPT**：✅（这张表是提案成本论据的骨架）

---

## 3. 一致性与可控性技术（比画质更关键）

### [E-A-25] VACE（阿里，"万能视频编辑器"）—— 参考图/区域/深度/姿态统一控制
- **事实**：arXiv:2503.07598「VACE: All-in-One Video Creation and Editing」，发布 2025-03-10，为 Wan 系列官方 README 中列出的官方衍生模型之一（`https://github.com/ali-vilab/VACE`，Wan2.1 发布日 2025-05-14 公告引入）。
- **来源类型**：论文原文 / 官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://arxiv.org/abs/2503.07598
  - 仓库: https://github.com/ali-vilab/VACE
  - 发布/更新日期: 2025-03-10
- **我的推断**：**开源侧目前唯一把"参考图 / 局部区域 / 深度图 / 姿态"等控制信号统一进一个模型的工作**，是自部署路线做可控性的核心组件。（具体控制类型清单以论文原文为准，本文未逐项核实。）
- **可用于 PPT**：✅（作为"开源可控性代表"）

### [E-A-26] Kling-MotionControl —— 驱动视频→参考角色的整体动作迁移
- **事实**：arXiv:2603.03160「Kling-MotionControl Technical Report」，发布 2026-03-03。官方摘要：「a unified **DiT-based** framework engineered specifically for robust, precise, and expressive **holistic character animation**. Leveraging a **divide-and-conquer** strategy … orchestrates heterogeneous motion representations tailored to the distinct characteristics of **body, face, and hands**, effectively reconciling large-scale structural stability with fine-grained articulatory expressiveness」，并通过 **adaptive identity-agnostic learning** 实现跨身份泛化（从写实真人到风格化卡通）。
- **来源类型**：论文原文
- **证据等级**：A
- **来源**：
  - URL: https://arxiv.org/abs/2603.03160
  - 发布/更新日期: 2026-03-03
- **支撑细节**：摘要中的"身体/脸/手分离建模"是对**口播数字人最直接相关**的技术描述——手部与面部口型是需要单独保障的两类运动。
- **我的推断**：**这是"动作可控"这一维度的当前最强公开技术证据**，且由可灵官方发布、可经 Runway 等渠道触达（见 E-A-08）。对"房产中介数字人按真人动作/表情复现"这类需求，优先级应高于纯画质模型。
- **可用于 PPT**：✅

### [E-A-27] Kling-Omni —— 生成/编辑/推理统一的多模态框架
- **事实**：arXiv:2512.16776「Kling-Omni Technical Report」，发布 2025-12-18。官方摘要：「a generalist generative framework designed to synthesize high-fidelity videos directly from **multimodal visual language inputs**」，采用端到端视角，「bridges the functional separation among diverse **video generation, editing, and intelligent reasoning** tasks, integrating them into a holistic system」，统一处理**文本指令、参考图、视频上下文**为单一多模态表示。
- **来源类型**：论文原文
- **证据等级**：A
- **来源**：
  - URL: https://arxiv.org/abs/2512.16776
  - 发布/更新日期: 2025-12-18
- **我的推断**：**"生成 + 编辑 + 推理"合并到一个模型**，意味着工作流从"多工具串联"变为"单模型多任务"，对实习团队的工程复杂度是显著利好。
- **可用于 PPT**：✅

### [E-A-28] Kling-Avatar —— 多模态指令驱动的长时长口播数字人
- **事实**：arXiv:2509.09595「Kling-Avatar: Grounding Multimodal Instructions for Cascaded Long-Duration Avatar Animation Synthesis」，发布 2025-09-11。官方摘要指出既有方法「treat instruction conditioning merely as **low-level tracking** driven by acoustic or visual cues, **without modeling the communicative purpose** conveyed by the instructions」，导致叙事连贯性与角色表现力受损；Kling-Avatar 采用**两阶段级联**：第一阶段用 MLLM「director」产出 blueprint video 统筹角色动作与情绪等高层语义，第二阶段依 blueprint 关键帧生成后续片段。
- **来源类型**：论文原文
- **证据等级**：A
- **来源**：
  - URL: https://arxiv.org/abs/2509.09595
  - 发布/更新日期: 2025-09-11（v2）
- **我的推断**：**这是与本项目（AI 口播数字人）命题最贴近的一篇官方论文**——"用多模态 LLM 做导演、决定高层语义与情绪，再交给视频模型出画面"正是口播数字人产品化的合理架构。可直接作为提案技术路线的引用。
- **可用于 PPT**：✅✅（优先级最高，建议作为技术路线图的锚点）

### [E-A-29] LTX-2 的开源控制手段（关键帧 / IC-LoRA）
- **事实**：LTX-Video 官方 README 对 LTX-2 的能力列出：「**Multiple keyframe support**」「**IC-LoRA control models** for precise generation」「Standard **LoRA** support for style customization」「**Latent upsampler** for multiscale pipelines」，且 LTX-2「has been **built into ComfyUI core**」。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Lightricks/LTX-Video （README 的 LTX-2 章节）
  - 查询日期: 2026-09-29
- **我的推断**：**"多关键帧 + IC-LoRA + ComfyUI core 内置"是开源侧可控性最完整的一套现成工具链**，且不需要自研训练代码。若走自部署，这是投入产出比最高的功能组合。
- **可用于 PPT**：✅

### [E-A-30] Wan2.2-Animate-14B / S2V-14B —— 开源侧的角色动画与语音驱动
- **事实**：Wan2.2 官方 README 记录：**Wan2.2-Animate-14B**（2025-09-19 发布）是「an unified model for **character animation and replacement** with holistic **movement and expression replication**」，已集成 Diffusers（2025-11-13）；**Wan2.2-S2V-14B**（2025-08-26 发布）是「an **audio-driven** cinematic video generation model」，含推理代码、权重与技术报告。此外社区项目 **TalkVerse**（基于 Wan2.2-TI2V-5B）实现「**minute-long** audio-driven human video generation」。
- **来源类型**：官方仓库
- **证据等级**：A（Animate/S2V 为官方）；B（TalkVerse 为社区项目）
- **来源**：
  - URL: https://github.com/Wan-Video/Wan2.2
  - URL: https://humanaigc.github.io/wan-animate （Animate 项目页）
  - URL: https://humanaigc.github.io/wan-s2v-webpage （S2V 项目页）
  - 查询日期: 2026-09-29
- **我的推断**：**开源侧已具备"语音→人物视频"（S2V）与"角色动作/表情复现"（Animate）两块拼图**，加上 VACE 的参考控制与 LTX-2 的 LoRA/关键帧，**自部署路线在功能清单上并非空白**——真正的约束回到吞吐与许可（见 E-A-16、E-A-24）。
- **可用于 PPT**：✅

### [E-C-32] "Step-Video-Audio" / "O-Mini" —— 未定位到原始出处
- **事实**：任务 brief 要求核查的 **Step-Video-Audio** 与 **O-Mini** 两个音频同步生成模型，本次**未能在 arXiv 检索到对应论文**（`ti:"Step-Video-Audio"`、`all:"Step-Video-Audio"` 均无结果；`ti:"Omni" AND abs:"audio-driven"` 返回的是 Vorch-Omni、DreamID-Omni 等**其他论文**）。相近但**不能等同**的已核实工作为：Kling-Foley（arXiv:2506.19774，2025-06-24，video-to-audio 方向的扩散 Transformer）、Step-by-Step Video-to-Audio Synthesis via Negative Audio Guidance（arXiv:2506.20995，2025-06-26）。
- **来源类型**：无
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: 未定位
  - 发布/更新日期: 未标注
- **我的推断**：**"Step-Video-Audio" 很可能是二手资料中把 Step-Video 系列与音频能力混称的产物**，或指代一篇非 arXiv 的技术报告。**不得写入 PPT**，除非找到原始出处。
- **可用于 PPT**：❌

---

## 4. AI 视频的实际生产工作流与成本

### [E-A-33] "智能分镜"已进入 Kling v3 官方 API 参数层（storyboard → shot 的一手证据）
- **事实**：阿里云百炼官方文档在可灵 v3 接口下列出四类能力：**文生视频**、**「文生视频（智能分镜）」**、**参考生视频（`type=feature+first_frame`）**、**视频编辑**。「智能分镜」为官方独立接口，即"先分镜、再逐镜生成"已被产品化为可编程能力，而非从业者土法。
- **来源类型**：官方技术文档
- **证据等级**：A
- **来源**：
  - URL: https://help.aliyun.com/zh/model-studio/kling-video-generation-api-reference
  - 发布/更新日期: 2026-09-04
- **我的推断**：**本项目的"房产讲解视频"本质就是分镜流水线**（开场 → 小区环境 → 户型讲解 → 配套 → 结尾 CTA）。既然分镜已成官方 API 能力，**不需要自研 storyboard 编排**，可直接调用，这会显著压缩实习团队的工程量。
- **可用于 PPT**：✅✅（直接决定技术架构）

### [E-A-34] 参考生视频的宽高比规则（工程细节，实操必踩）
- **事实**：阿里云文档原文规定宽高比规则：**图生视频-基于首帧**：以首帧的宽高比为基准，无需填写；**图生视频-基于首尾帧**：以首帧的宽高比为基准；**参考生视频（`type=feature+first_frame`）**：以首帧的宽高比为基准；**视频编辑**：以输入视频的宽高比为基准。`type=base` 时输出视频时长与传入视频时长相同，此时 `duration` 参数无效，按输入视频时长四舍五入取整计费。
- **来源类型**：官方技术文档
- **证据等级**：A
- **来源**：
  - URL: https://help.aliyun.com/zh/model-studio/kling-video-generation-api-reference
  - 查询日期: 2026-09-29
- **我的推断**：**"首帧决定画幅"意味着必须先定好构图与比例（房产场景通常需 9:16 竖版），再谈生成**。这是排期上容易被漏掉的前置工作。
- **可用于 PPT**：⚠️（工程细节，适合技术附录）

### [E-A-35] Runway 的 credits 滚存与无水印（成片阶段工序的取舍依据）
- **事实**：Runway 官方定价页显示：Max 层「**Credits roll over 1 mo.**」；Pro 层起「**No watermarks**」「**4K upscaling**」「Music, voice & sound effects generation」；Pro 层含「Top-up credits anytime」；Max 层含「Studio-grade quality generations: **HDR, ProRes, image sequences**」与「**Topaz AI Upscale** Unlimited」。各层并行能力不同（Standard 5 路、Pro 15 路、Max 20 路视频并行）。
- **来源类型**：官方定价页
- **证据等级**：A
- **来源**：
  - URL: https://runwayml.com/pricing
  - 查询日期: 2026-09-29
- **我的推断**：**建议工作流：生成用可灵/Seedance（分镜+多参考+按秒计费），成片收尾（无水印、4K 升采样、HDR/ProRes 输出）用 Runway。** 这样既拿到可控性，又避开 4K 升分的高昂生成成本。
- **可用于 PPT**：✅

### [E-A-36] 开源自部署的工具链生态已成熟（ComfyUI / Diffusers / DiffSynth-Studio）
- **事实**：可核实的官方集成记录：Wan2.2 的 T2V/I2V/TI2V 已集成 **Diffusers**（2025-07-28），Wan2.2-Animate-14B 集成 Diffusers（2025-11-13）；**DiffSynth-Studio**（ModelScope）提供 Wan 2.2 的逐层 offload、**FP8 量化**、序列并行、LoRA 与全量训练；**HunyuanVideo-1.5** 官方仓库自带 **ComfyUI** 集成并提供 **LightX2V** 加速入口；**LTX-2 已内置 ComfyUI core**；**Step-Video** 有官方社区实现 `ComfyUI-StepVideo`（42 stars，MIT）。
- **来源类型**：官方仓库
- **证据等级**：A
- **来源**：
  - URL: https://github.com/Wan-Video/Wan2.2
  - URL: https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5
  - URL: https://github.com/Comfy-Org/ComfyUI （ComfyUI 本体）
  - 查询日期: 2026-09-29
- **我的推断**：**自部署的"工程不熟"风险已大幅下降**——ComfyUI + Diffusers 是通用件，不必从零搭推理服务。但这不改变 E-A-16 的吞吐劣势。
- **可用于 PPT**：✅

### [E-A-37] 生成 1 分钟成片素材的成本推演（全部基于官方单价，明确标注为推断）
- **事实**：以下单价均来自本次查询的官方定价页：Veo 3.1 Lite 720p $0.05/s、Fast 720p $0.10/s、Standard 720p $0.40/s（E-A-02）；Gemini Omni Flash ≈$0.10/s 720p（E-A-03）；Runway Gen-4.5 = 12 credits/s，Max 层 9500 credits / $76（E-A-07）。可灵按秒计费但单价在控制台、未取到（E-B-10）。
- **来源类型**：官方定价页（汇总）
- **证据等级**：B（因其中可灵、Seedance、Sora 单价缺失，仅能给出部分区间）
- **来源**：
  - URL: https://ai.google.dev/gemini-api/docs/pricing ；https://runwayml.com/pricing
  - 查询日期: 2026-09-29
- **支撑细节（推断，数学可复核）**：
  - Veo 3.1 Lite 720p：$0.05 × 60 = **$3.00 / 分钟**
  - Veo 3.1 Fast 720p 或 Gemini Omni Flash：$0.10 × 60 = **$6.00 / 分钟**
  - Veo 3.1 Standard 720p：$0.40 × 60 = **$24.00 / 分钟**
  - Runway Gen-4.5（Max 层单价）：($76 / 9500) × 12 × 60 ≈ **$5.76 / 分钟**
- **我的推断**：**以上为"理想 1 次成功生成"的理论下限。** 真实成本必须乘以抽卡系数 N。⚠️ **本次未能取到任何可核验的行业平均抽卡系数或"生成 1 分钟成片实际花费"的一手实测数据**，因此**不给出含 N 的最终成本数字**。若 N=5（纯假设），上述区间放大为 **$15–120 / 分钟素材**。**这一步必须靠实习期真实抽卡统计补齐，不能在提案里编。**
- **可用于 PPT**：⚠️ 仅可展示理论下限区间，且必须标注"不含抽卡重试，实测待补"

### [E-C-38] 从业者工作流实践与"1 分钟成片成本实测" —— 未取到可核验一手来源
- **事实**：本次调研**未定位到**任何同时满足"公开可核验 URL + 明确测试条件 + 可复核成本数字"的一手工作流实录或成本实测（搜索结果中的社区/自媒体/聚合平台内容依 `_规范.md` 归为 C 级且多含营销动机，不予采信）。
- **来源类型**：无
- **证据等级**：C ⚠️ 未核验
- **来源**：
  - URL: 未定位
  - 发布/更新日期: 未标注
- **我的推断**：E-A-33（智能分镜官方接口）+ E-A-35（Runway 成片工序）+ E-A-36（ComfyUI 生态）已能**重构出一条有官方依据的工作流**，建议 PPT 用这三条拼图替代无出处的"行业普遍做法"叙述。
- **可用于 PPT**：❌

---

## 5. 版权与合规

### [E-A-39] 《生成式人工智能服务管理暂行办法》—— 适用范围与训练数据义务
- **事实**：该办法由国家互联网信息办公室等七部门公布（网信办令第 15 号），2023-05-23 审议通过，**自 2023-08-15 起施行**。关键条款：
  - **适用范围**：利用生成式 AI 技术**向中华人民共和国境内公众提供**生成文本、图片、音频、**视频**等内容的服务适用本办法；**"行业组织、企业、教育和科研机构……研发、应用生成式人工智能技术，未向境内公众提供生成式人工智能服务的，不适用本办法的规定"**。
  - **第四条**：应遵守法律行政法规，**（三）尊重知识产权、商业道德，保守商业秘密**，不得利用算法、数据、平台等优势实施垄断和不正当竞争；**（四）不得侵害他人肖像权、名誉权、荣誉权、隐私权和个人信息权益**。
  - **第七条**：训练数据处理须「（一）使用具有**合法来源**的数据和基础模型；（二）涉及知识产权的，**不得侵害他人依法享有的知识产权**；（三）涉及个人信息的，应当取得**个人同意**……」。
- **来源类型**：监管文件（国家网信办官网原文）
- **证据等级**：A
- **来源**：
  - URL: http://www.cac.gov.cn/2023-07/13/c_1690898327029107.htm
  - 发布/更新日期: 2023-07-13（署 2023 年 7 月 10 日）
- **我的推断**：**"未向境内公众提供则不适用"是本项目最关键的一条**——若数字人仅供企业内部使用（如中介自己看房源讲解稿），合规义务大幅降低；一旦做成对外发布的小程序/短视频，则全套义务生效。**这决定了产品形态选择，必须在提案里明确。**
- **可用于 PPT**：✅✅（决定产品边界）

### [E-A-40] 《人工智能生成合成内容标识办法》—— AI 视频必须打标（易被漏掉的硬要求）
- **事实**：**国信办通字〔2025〕2 号**，国家互联网信息办公室、工业和信息化部、公安部、国家广播电视总局联合发布，**自 2025-09-01 起施行**（发布日 2025-03-14，署 2025-03-07）。与 AI 视频直接相关的强制要求原文：
  - 「人工智能生成合成内容标识包括**显式标识**和**隐式标识**。」
  - 显式标识（视频）→「**（四）在视频起始画面和视频播放周边的适当位置添加显著的提示标识**，可以在视频末尾和中间适当位置添加显著的提示标识」
  - 「服务提供者提供生成合成内容**下载、复制、导出**等功能时，应当确保文件中含有满足要求的**显式标识**。」
  - 隐式标识→「应当在生成合成内容的**文件元数据**中添加隐式标识，隐式标识包含**生成合成内容属性信息、服务提供者名称或者编码、内容编号**等制作要素信息。」「鼓励……添加**数字水印**等形式的隐式标识。」
  - 传播平台须核验文件元数据中的隐式标识，并据此分级添加「属于 / 可能为 / 疑似」生成合成内容的提示标识。
  - 用户可申请**不含显式标识**的内容，但须通过用户协议明确标识义务与使用责任，并**留存日志不少于六个月**。
  - 第十一条：还须符合**强制性国家标准**的要求。
- **来源类型**：监管文件（国家网信办官网原文）
- **证据等级**：A
- **来源**：
  - URL: https://www.cac.gov.cn/2025-03/14/c_1743654684782215.htm
  - 国务院网转载: https://www.gov.cn/zhengce/zhengceku/202503/content_7014286.htm
  - 发布/更新日期: 2025-03-14
- **支撑细节**：配套**强制性国家标准**《网络安全技术 人工智能生成合成内容标识方法》（市场监管总局/国家标准委 2025 年第 3 号公告批准发布），与《标识办法》**同步于 2025-09-01 实施**（据工业和信息化部政务页面转述，https://wap.miit.gov.cn/xwfb/mtbd/wzbd/art/2025/art_5e46c60f9a7141cdb584eb139f476ce9.html ，2025-03-15）——**国标原文本次未直接核实，按 B 级对待。**
- **我的推断**：**这是本项目最容易被漏掉、且一旦漏掉就属违规的工程项。** 方案里必须包含：① 视频起始画面烧录提示标识；② 导出文件写入元数据隐式标识（属性/服务方/内容编号）。好消息是这两项是**纯工程活，不依赖模型能力**，应列入实习任务清单而非"待研究"。
- **可用于 PPT**：✅✅（建议单列一页合规页）

### [E-A-41] 美国版权局：AI 生成物本身不受版权保护，但人创部分受保护
- **事实**：美国版权局（USCO）《Copyright and Artificial Intelligence, Part 2: Copyrightability Report》于 **2025-01-29** 发布（Part 1 数字复制品报告发布于 2024-07-31；Part 3 拟处理训练环节的法律问题）。官方结论（NewsNet Issue 1060）：「The Office confirms that **the use of AI to assist in the process of creation or the inclusion of AI-generated material in a larger human-generated work does not bar copyrightability**. It also finds that **the case has not been made for changes to existing law to provide additional protection for AI-generated outputs**.」即：纯 AI 生成部分不享有版权，但人类创作性贡献（如脚本、分镜、剪辑编排）可受保护。
- **来源类型**：监管文件 / 政府机构公告
- **证据等级**：A
- **来源**：
  - 报告原文: https://www.copyright.gov/ai/Copyright-and-Artificial-Intelligence-Part-2-Copyrightability-Report.pdf
  - 官方摘要: https://www.copyright.gov/newsnet/2025/1060.html
  - 发布/更新日期: 2025-01-29
- **我的推断**：**实务含义：若成片要登记版权并主张独占，必须保留完整的人类创作链路证据**（脚本→分镜→选材→剪辑工程文件）。这恰好也解释了 E-A-33 中"用官方分镜能力"的价值——分镜由人定义，才构成人类的创造性贡献。
- **可用于 PPT**：✅

### [E-A-42] 训练数据与商用授权风险：三个层级的来源核对结论
- **事实**：本次就"训练数据争议/商用授权风险"做了三层核对：
  1. **开源侧**：Wan 系列为 Apache-2.0，但**许可证只授权权重使用，并不担保训练数据已获授权**；腾讯社区许可额外要求遵守其 Acceptable Use Policy 与地域限制；LTX-2.x 对年营收 ≥$10M 实体强制付费许可。
  2. **API 侧**：可灵企业会员页明确表述「创作成果归属企业，**绝不挪用企业数据训练模型**」；Sora 2 侧微软文档记载 Azure OpenAI 侧提供输入输出审核与内容过滤（"Sora 2 blocks all IP and photorealistic content"）。
  3. **监管侧**：第七条训练数据"合法来源"义务由**服务提供者**承担，**不等于使用者被免责**。
- **来源类型**：官方仓库 / 公司官网 / 监管文件（综合）
- **证据等级**：B（综合判断，非单一来源）
- **来源**：
  - URL: https://github.com/Wan-Video/Wan2.2 ；https://www.cac.gov.cn/2023-07/13/c_1690898327029107.htm ；https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/video-generation ；https://klingai.com/dev/pricing
  - 查询日期: 2026-09-29
- **我的推断**：**风险最高的是"自部署"而非"调 API"**——因为调 API 时训练数据合规责任在厂商侧且有企业条款背书，自部署则由使用方继承全部数据来源风险。**这一点与常见的直觉相反，应在提案中明确写出。**
- **可用于 PPT**：✅

### [E-A-43] USCO Part 3（训练环节）发布状态 —— 未确认
- **事实**：Library of Congress 新闻稿（2025-01-29）称「The final, forthcoming **Part 3** will address the legal implications of **training AI models on copyrighted works**, including licensing considerations and the allocation of any potential liability.」截至 2026-09-29 本次查询，**未核实 Part 3 是否已正式发布**。
- **来源类型**：政府机构公告（对 Part 3 状态的陈述）
- **证据等级**：C ⚠️ 未核验（针对 Part 3 的当前状态）
- **来源**：
  - URL: https://newsroom.loc.gov/news/copyright-office-releases-part-2-of-artificial-intelligence-report/s/f3959c36-d616-498d-b8f9-67641fd18bab
  - 发布/更新日期: 2025-01-29
- **我的推断**：训练数据的确权责任分配是"商用授权风险"的核心变量，**Part 3 的结论会实质改变风险评估**。列为提案前必须复查项。
- **可用于 PPT**：❌（待 Part 3 发布后重评）

---

## 待核验清单

### 高优先级（阻塞提案结论，务必先补）

| # | 待核验事项 | 现状 | 建议动作 |
|---|---|---|---|
| 1 | **可灵 Kling v3 官方单价（元/秒，各档）** | 官方定价页 JS 渲染，未取到 | 用可访问网络打开 `klingai.com/dev/pricing`（登录态）或阿里云百炼控制台价格页截图留档 |
| 2 | **Sora 2 官方 API 单价** | `openai.com` 与 `platform.openai.com` 均被 Cloudflare 拦截 | 换网络环境或用 Azure OpenAI 定价页取"按秒计费"单价 |
| 3 | **Seedance 2.0 官方 API 单价（火山方舟）** | 未取得官方定价原文 | 查火山引擎方舟（Volcengine Ark）模型广场 `doubao-seedance-2-260128` / `-2-fast-260128` 价格 |
| 4 | **Sora 消费端产品是否已关停** | 搜索摘要称"no longer available"与"2026 关停"，互相矛盾且均未核实 | 打开 `openai.com/index/sora-2/` 原文确认 |
| 5 | **生成 1 分钟成片素材的实际成本（含抽卡系数 N）** | 只有官方单价（理论下限），无实测 N | **实习期自己抽卡统计**：同一分镜 20 次，记录可用率与实际花费 |

### 中优先级（影响选型完整性）

| # | 待核验事项 | 现状 |
|---|---|---|
| 6 | Midjourney Video / Luma Dream Machine / Pika 的最新版本号与定价 | 官方页 JS 渲染，均未取到；建议 PPT 中直接不列 |
| 7 | USCO AI 报告 **Part 3**（训练环节）是否已发布 | 未确认；若已发布须重评训练数据风险 |
| 8 | 强制性国标《网络安全技术 人工智能生成合成内容标识方法》原文 | 仅从工信部转述得知同步实施，未读国标原文；须确认元数据字段的技术要求 |
| 9 | 《互联网信息服务深度合成管理规定》第十七条第一款的具体情形 | 《标识办法》引用但本文未展开核对 |
| 10 | "Step-Video-Audio" / "O-Mini" 的原始出处 | arXiv 检索无结果，疑为二手资料误称；**核实前不得写入 PPT** |
| 11 | Kling 3.0 / Seedance 2.5 的官方首发公告 | 仅有 Runway 定价页侧证，需回可灵/字节官方渠道确认版本与日期 |
| 12 | 微信/抖音等平台对 AI 生成房产内容的水印与推荐政策 | 完全未调研，但直接影响投放可行性 |

### 已知方法论风险（记录以免重犯）

- **arXiv 编号不可凭记忆引用**：本次凭记忆构造的 4 个 ID 全部指向无关领域论文，经 `export.arxiv.org/api/query` 逐条核验后剔除。后续所有论文引用必须过 API。
- **第三方"Veo 4"类 SEO 内容不可信**：多个站点（veo3ai.io、ourdream.video 等）互相矛盾地宣称 Veo 4 已发布，官方 DeepMind 模型页与 Gemini 定价页均无此条目。**凡官网无对应条目者，一律不得写入 PPT。**
- **"开源"≠"可商用"**：8 个开源视频模型中仅 4 个为无条件可商用（Apache-2.0 / MIT），其余有法域限制、营收门槛或许可证缺失。选型表必须逐个核许可证原文，不能看 GitHub 的 license 标签了事（腾讯与 LTX 均为 `NOASSERTION`）。

---

## 附：本次调研的核验方法说明（可复现）

- **GitHub 元数据**：`https://api.github.com/repos/{owner}/{repo}`（star 数、`license.spdx_id`、`pushed_at`），查询日 2026-09-29。
- **许可证原文**：`https://raw.githubusercontent.com/{owner}/{repo}/{branch}/LICENSE*`，逐份读取正文（Wan2.2 的 `LICENSE.txt`、HunyuanVideo-1.5 的 `LICENSE`、LTX-2 的 `LICENSE-2_x`）。
- **arXiv 元数据**：`http://export.arxiv.org/api/query`，逐条核对标题与 `published`（注意：feed 级 `<updated>` 会返回查询当日，不可作为论文更新时间）。
- **官方定价**：直接读取定价页 HTML（Google Gemini API 定价页为服务端渲染，可提取；Runway 定价页可提取；OpenAI 与可灵为 JS 动态渲染，未取到）。
- **监管原文**：`cac.gov.cn` 与 `copyright.gov` 官网直读全文。

---

## 附录：核心论文出处表（经 arXiv 官方 API 逐条核实）

> 用途：PPT 技术附录的引文出处。全部以 `http://export.arxiv.org/api/query` 核对标题与 `published` 日期，查询日 2026-09-29。
> ⚠️ 注意：arXiv feed 的 `<updated>` 会返回查询当日，**不可**当作论文更新时间使用。

| arXiv ID | 标题 | 发布日期 | 对应条目 |
|---|---|---|---|
| 2503.20314 | Wan: Open and Advanced Large-Scale Video Generative Models | 2025-03-26 | E-A-15 / E-A-16 |
| 2412.03603 | HunyuanVideo: A Systematic Framework For Large Video Generation Model | 2024-12-03 | E-A-18 |
| 2511.18870 | HunyuanVideo 1.5 Technical Report | 2025-11-24 | E-A-17 |
| 2408.06072 | CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer | 2024-08-12 | E-A-22 |
| 2501.00103 | LTX-Video: Realtime Video Latent Diffusion | 2024-12-30 | E-A-20 |
| 2502.10248 | Step-Video-T2V Technical Report | 2025-02-14 | E-A-23 |
| 2503.07598 | VACE: All-in-One Video Creation and Editing | 2025-03-10 | E-A-25 |
| 2509.09595 | Kling-Avatar: Grounding Multimodal Instructions for Cascaded Long-Duration Avatar Animation Synthesis | 2025-09-11 | E-A-28 |
| 2506.19774 | Kling-Foley: Multimodal Diffusion Transformer for High-Quality Video-to-Audio Generation | 2025-06-24 | E-C-32（旁证） |
| 2506.20995 | Step-by-Step Video-to-Audio Synthesis via Negative Audio Guidance | 2025-06-26 | E-C-32（旁证） |
| 2512.16776 | Kling-Omni Technical Report | 2025-12-18 | E-A-27 |
| 2603.03160 | Kling-MotionControl Technical Report | 2026-03-03 | E-A-26 |
| 2604.14148 | Seedance 2.0: Advancing Video Generation for World Complexity | 2026-04-15 | E-A-11 / E-A-12 |

### 核验过程记录（方法论教训）

本次曾凭记忆构造 4 个 arXiv ID（`2504.20925`、`2503.20213`、`2506.09931` 等），经 API 核实**全部指向完全无关领域的论文**（分别为"微泳体在 Brinkman 流中的化学趋向聚集动力学""拓扑绝缘体/铁磁体中的非互易自旋-电荷转换""单载波 ISAC 的超奈奎斯特信号"）。这些错误编号已全部剔除，未写入正文。

**结论：AI 视频领域的 arXiv 编号无法凭记忆引用，必须逐条过 `export.arxiv.org/api/query` 核实标题与日期。** 后续任何新增论文条目沿用此流程。
