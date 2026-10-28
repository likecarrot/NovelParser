# 项目与数据契约（版本 6）

`project.json` 由 helper 创建和更新，位于各分卷制作子目录中，包含 schema_version（6）、title（书名）、volume（卷名，如“第1卷”）、source、script、reviews、script_lock、art、layout、exports、final_review。书名顶层目录维护全书 `README.md`、`docs/` 模块化介绍文档、集中归档的 `source_texts/` 与切分后的 `split_texts/`。以当前卷为独立制作与锁定单位；其他卷尚未编剧、审查或完成，不阻塞当前卷制作与交付。不要直接更改 source、锁、尝试号或完成记录来放行。编剧时在当前卷状态目录的 `scripts/` 中写完整 script JSON，通过 set-script 导入；报告保存到 `reports/`。统一项目分卷布局见 [commands.md](commands.md)，原文新版本的内部状态目录见 [recovery.md](recovery.md)。统一采用单一权威标准 schema_version=6。

## Source

files 记录原始输入绝对路径、项目内 archive_path、格式、编码和 SHA256；制作使用归档副本，外部原稿变化单独警告；chapters 使用顺序 ID（不以原章号为唯一键），保留标题、正文存在性、阅读记录；units 记录 id、chapter_id、kind、text、locator。TXT 用行号，DOCX 用段落/注释位置，PDF 用页与行，EPUB 用 spine/member/段落。

集中归档在 `source_texts/` 下生成 `archive_manifest.json`，切分原稿在 `split_texts/` 下生成 `split_manifest.json`。同名原稿自动附加序号与来源映射，切分记录各段落指纹与范围，防止覆盖与漏文。

读取 `chapter` 的输出获取真实 unit ID，不猜编号。所有正文 unit 均需映射，章标题本身不用绘制。source_index_hash 覆盖不可变来源索引，阅读笔记不改变它。issues 需要逐项查明；空章可通过注明确实无正文解决，不代表补画该章。

## Script

使用 assets/script-template.json 的结构。以下为必填字段：

| 对象 | 字段与要求 |
|---|---|
| 本卷 | outline：本卷结构与核心故事梗概（必须 100% 严格提炼自已确认正文，绝对禁止脑补编造）；ending：本卷实际原文真实收尾 |
| style | genre、look、palette、selection_reason、reading_direction=ltr/rtl；art_direction 记录线条、明暗、色彩、背景等文字规范。不写画幅、字高、字体与参考图 |
| character | id、name、aliases 数组、importance=major/supporting/minor；narrative 含 goal/motivation/voice/arc；visual 必须含 face_shape/eyes/brows/nose_mouth/hair/body/posture/age 八项且每项可渲染，补全项写入 visual.invented_fields；base_portraits 每个形态版本一张中性基准图；derived_looks 完整造型描述；source_facts、design_notes 数组 |
| source_fact | text 与非空 source_unit_ids；必须为原文真实事实，设计注记不能写成原作事实 |
| setting | id、description、space、light、palette、material、fixtures、time；原文无依据的补全项写入 setting.invented_fields |
| event | id、description（严格对应原著真实事件，绝不捏造新情节）、非空 source_unit_ids；每个事件必须在画格中出现 |
| scene | id、chapter_id、setting_id；可补时刻、叙事层级、`reader_knows` / `character_knows`（读者与角色此刻知道什么）、`nsfw_level`（0-3） |
| panel | id、chapter_id、scene_id、source_unit_ids、event_ids、cast、action、shot、space、expression、scene_text、state_before、state_after、dialogue（台词对白/旁白必须严格忠实于原著语义，严禁私自编造加戏）；可补 visual_plan 对象记录视觉中心、层次与局部色彩等；可补 `nsfw_level`（0-3，覆盖所在场景的级别）；可补 `looks`（角色 ID → 衍生图 ID，缺省用默认造型）；可补 `extension_of` / `extension_kind` / `progress` / `new_information` 组成延长格 |
| page | id、chapter_id、按阅读顺序排列的 panel_ids；rows 可选，仅表达阅读顺序。页内格数不设上限 |
| continuity_handover | opening_state 与 closing_state，记录本卷开场继承与结末状态（人物状态、世界规则、未回收伏笔），供跨卷交接审查核验 |
| source_disposition | unit_id、kind=context/repetition/paratext、reason；context 还需真实 panel_ids |
| chapter_adaptation | 每个正文章恰好一条：chapter_id、document_path、document_sha256、details、unit_audits；具体字段与呈现证据见 [detail-records.md](detail-records.md) |

panels 数组就是本卷镜头顺序；pages 按此顺序覆盖每格恰好一次，不能遗漏、重复或改变顺序。相同章号的不同卷使用不同内部 chapter_id。所有分镜内容严格忠于原著，严禁胡编乱造。

### 基准图与衍生图

角色图分三层。技能只输出文字，不生成图片：

1. **基准图**（`base_portraits`）：每个形态版本一张，只描述身体与面部，不含服装、道具、场景、情绪。用于文生图得到标准像。
2. **衍生图**（`derived_looks`）：完整重述基准特征，再写服装与状态变化。基于基准图做图生图。
3. **分镜图**：在某个衍生图之上生成，由 `panel.looks` 指定。

```json
{
  "appearance_versions": [{"id": "base"}],
  "base_portraits": [{
    "id": "base",
    "description": "成年男性，中等身高…（至少 80 字，只写身体与面部）",
    "framing": "正面全身，自然站姿，纯色中性背景，均匀正面光",
    "excludes": "服装、道具、场景、情绪、文字"
  }],
  "derived_looks": [{
    "id": "white_coat",
    "label": "白大褂",
    "form": "base",
    "is_default": true,
    "description": "成年男性，中等身高…（重述基准特征），穿白大褂…"
  }]
}
```

规则：

- 基准图 id 必须与形态版本 id 一一对应。身体层面发生根本变化（年龄跨度、变身、毁容、长期伤残）才新增形态版本与基准图；换装、换发型、脏污、可恢复伤势属于衍生图。
- 基准图不得出现服装词或情绪词，出现即报错。
- 衍生图必须写完整描述，不能只写「身着华服」这样的增量。
- 每个形态版本至少一个衍生图，且恰好一个 `is_default`。
- 分镜用 `looks` 指定造型；不写则用默认造型。

### 分镜自包含

每个 `scene_text` 必须能单独读懂，不依赖前后格：

- `cast` 中的每个角色都要在描述里点名，不能用「他」「她」
- 不使用「眼神空洞」「目光空洞」这类脱离上下文就失真的短语，改写为可见线索

```json
"scene_text": "白玲坐在办公桌前低头翻书，眼睛半睁，视线落在纸页上，眼白略多。"
```

### 硬性特征与补全

人物和环境都有必须写满的特征清单。原文没有描写时不能留空，也不能写「原文未明确」，必须补一个符合设定的值，并在 `invented_fields` 里声明。

人物 `visual` 八项：`face_shape`、`eyes`、`brows`、`nose_mouth`、`hair`、`body`、`posture`、`age`。

场景六项：`space`、`light`、`palette`、`material`、`fixtures`、`time`。

```json
{
  "visual": {
    "face_shape": "窄长脸，颧骨略高",
    "eyes": "细长眼，眼尾微垂",
    "invented_fields": ["eyes", "face_shape"]
  }
}
```

规则：

1. 每项都要能直接画出来，「很有故事感」这类不算。
2. 出现「未明确」「未提及」「不详」「待定」等占位词，`check-script` 直接报错。
3. 补全顺序：原文描写 → 同作品其他角色或场景的既有设定 → 题材与年代默认值。
4. 补全值一经确定，全书沿用，跨卷复用同一档案。
5. `invented_fields` 只能列清单内的字段，写错会报错。

### 读者与角色知情

`scene.reader_knows` 记录读者此刻已经知道的事实，`scene.character_knows` 按角色记录他们各自知道的。两者都是字符串数组，`character_knows` 的键必须是真实角色 ID。

用途是控制身份延迟揭晓：读者还不知道某人是谁时，画面描述只能用外观指称，不能提前写出真名或背景。

```json
{
  "reader_knows": ["保健室里有个穿白大褂的男人"],
  "character_knows": {"bai_ling": ["对方是主任"]}
}
```

两个字段都可选，一旦写了就必须是字符串数组。

### 画面描述（scene_text）

每格必须写 `scene_text`，一段 20 字以上的自然语言，说清这一格看得见的东西：谁在场、在哪里、做什么、什么表情、身上什么状态。

- 不写权重、画质词、LoRA、采样参数、画幅尺寸
- 不堆标签，用完整句子
- 多人同框写清左右、前后、接触点
- 这一段是交付物本身，不是给某个模型的提示词

### 内容分级

`nsfw_level` 表示该场景或该格的性内容强度，取整数 0–3：

| 级别 | 含义 |
|---|---|
| 0 | 无性内容 |
| 1 | 暗示、擦边，不含明确描写 |
| 2 | 明确的性描写，但克制 |
| 3 | 露骨描写 |

场景级别写在 `scene.nsfw_level`；同一场景内某格尺度不同时，用 `panel.nsfw_level` 覆盖。缺省视为 0。延长格只有在有效级别 ≥ 2 时才成立，且不得高于源格。

### 延长格

原文用「一直」「越来越」「停不下来」这类持续描述带过的一段行为，可以拆成多个观察点。这类格子标记为延长格：

```json
{
  "id": "p016_01",
  "extension_of": "p015_03",
  "extension_kind": "duration",
  "progress": 0.55,
  "new_information": "床脚接缝的铁片被震得跳起来，又被压回去"
}
```

- `extension_of`：源格 ID，必须指向真实存在、且自身不是延长格的分镜。
- `extension_kind`：`duration` 沿时间切、`detail` 切局部、`reaction` 切对方反应。
- `progress`：0–1，同一源格下按脚本顺序严格递增，不能重复。
- `new_information`：这一格比别的格多给了什么，不能为空，同一源格下不能重复。

硬约束：场景与源格相同、`cast` 是源格的子集、`source_unit_ids` 至少共享一条、`state_before` / `state_after` 与源格完全一致、有效 `nsfw_level ≥ 2`。数量不设上限，由编剧按叙事需要决定。

编剧工作按章递进：当前状态目录的 `docs/adaptation/<chapter_id>.md` 保存该章细节映射、逐场详细剧情剧本、格数依据、核查记录与章末交接；`scripts/chapters/<chapter_id>.json` 保存含 events/scenes/panels/pages/source_dispositions/chapter_adaptations 的详细章节草稿，相关人物/场景档案在本卷工作稿中维护。章节文件是工作材料，不另建 project.json 或伪造独立锁；初次编写直接保存草稿，用 `check-adaptation` 核对本章，再用 `set-script` 导入已汇总的工作稿。`set-script-chapter` 校验整卷结构，适用于完整结构稿上的单章更新，必须带对应章细节记录。最终关卡仍要求本卷全部章节与同版本审查。

### 来源引用与跨卷命名空间

`source_unit_ids` 支持本卷短字符串 ID（如 `"u0000001"`）与跨卷命名空间对象：
```json
{
  "volume_id": "vol-001",
  "source_index_hash": "64位SHA256指纹",
  "unit_id": "u0000002"
}
```
跨卷继承事实与设定时必须使用带命名空间的对象，严禁直接使用前卷短 ID 混入本卷。流水线会校验跨卷项目的 `source_index_hash` 与单元存在性，避免编号碰撞或静默失效。

art_direction 的字段及决策方法见 [art-direction.md](art-direction.md)；模板默认值需按本作调整。

rows 为画格 ID 数组的数组，展平后必须与 panel_ids 完全一致，只表达阅读顺序。不写 columns、aspect_ratio 或任何版面尺寸。

状态按角色 ID 保存对象，主字段固定：

| 字段 | 含义 |
|---|---|
| `form` | 形态版本，必须匹配 `appearance_versions` |
| `clothing` | 衣着，字符串或按部位的对象 |
| `exposure` | 暴露程度 |
| `condition` | 身体与精神状态 |
| `injuries` | 伤势，数组 |
| `held` | 持有物，数组 |
| `posture` | 姿态 |

非常规状态放进 `extension` 对象，不要新增顶层键。顶层出现未知字段会让 `check-script` 报错。

每个 cast 角色都必须有非空的 `state_before` 与 `state_after`。相邻出场时字段发生变化，要在当前格用 `state_transitions` 解释：

```json
{"character_id": "bai_ling", "fields": ["clothing"], "reason": "原文写明胸罩被扯开", "source_unit_ids": ["u0045"]}
```

没有解释的字段变化会导致 `check-script` 报错，`lock-script` 无法通过。

dialogue 为 `{kind,speaker,text,anchor?}`。kind 是 speech/thought/caption/sfx；speech/thought 的 speaker 必须在本格 cast 中。anchor 是 0..1 画面坐标，对应图中该说话人的位置；默认排版器使用其水平分量。文字原样保留，脚本自动添加说话人标识不改变对白正文。

ID 使用稳定、短且适合文件名的字母数字/连字符，避免路径符号。角色形式扩展、关系、道具与阶段设定可作为额外字段保存在相应对象；明确它们的来源与生效范围。

## Reports

审查/视觉报告不自动生成 pass。编剧或制作模型实际完成检查后保存 JSON；程序核验完整性与版本。

剧本报告：
```json
{
  "script_hash": "使用 check-script 输出的实际指纹",
  "reviewed_chapter_ids": ["全部有正文的实际章节 ID"],
  "checks": {"该审查类型的每个检查项": true},
  "evidence": "实际对照位置、发现、修订和复查结果",
  "issues": [{"severity":"major", "description":"具体问题", "resolved":true}]
}
```

coverage 检查项：all_source_read/events_preserved/arcs_preserved/ending_preserved（必须核对原著事件 100% 覆盖且零虚构事件）。
`events_preserved` 的语义检查须覆盖段内细节、事件过程、对白交锋和心理/情绪转折，不能只统计来源 ID 或事件 ID。逐章细节清单与分镜映射保存在当前状态目录的 `docs/adaptation/<chapter_id>.md`；结构化记录必须写入 script.chapter_adaptations，关卡核验来源引句、实际字段摘句、合并依据、逐段核查记录和文档指纹。coverage 的 evidence 引用清单路径、具体来源/画格、合并依据和实际复查结果；报告检查项保持原有字段，机械记录不替代语义检查。细则见 [narrative-density.md](narrative-density.md) 与 [detail-records.md](detail-records.md)。
continuity：causality/timeline/identity/states/knowledge_and_reveals（核对因果与状态逻辑 100% 来源于原著，严禁自设因果）。
comic：drawable_panels/dialogue_and_speakers/reading_order/pacing/text_density（核对分镜可绘制且台词/旁白忠于原著，严禁捏造台词）。

阶段审查报告（每5章阶段门禁报告，存入 `reports/stage_reviews/stage_XX.json`）：
```json
{
  "schema_version": 6,
  "stage_range": {
    "start_chapter_id": "ch000001",
    "end_chapter_id": "ch000005",
    "chapter_count": 5,
    "stage_script_hash": "绑定当前批次章节内容的指纹"
  },
  "stage_script_hash": "绑定当前批次章节内容的指纹",
  "reviewed_chapters": ["ch000001", "ch000002", "ch000003", "ch000004", "ch000005"],
  "stage_approved": true,
  "round_1_initial_audit": {
    "initial_verdict": "PASSED 或 REJECTED",
    "auditors": [
      {
        "auditor_id": "auditor_a",
        "conversation_id": "会话凭据",
        "scores": {"total_score": 85},
        "deductions": [{"reason": "扣分理由", "points": 10}],
        "issues": [
          {
            "id": "auditor_a_issue_01",
            "chapter_id": "ch000001",
            "source_quote": "小说实际引句",
            "problem_description": "问题说明",
            "suggested_fix": "修改建议"
          }
        ]
      }
    ]
  },
  "screenwriter_revisions": {
    "revision_summary": "整改综述",
    "applied_fixes": [
      {
        "issue_ref": "auditor_a_issue_01",
        "modified_panels": ["p001_01"],
        "before_revision": "修改前内容",
        "after_revision": "修改后内容（必须有差异）"
      }
    ],
    "adjudications": [
      {
        "issue_ref": "问题ID",
        "reason": "误报裁定理由（如原著后文已有说明）"
      }
    ]
  },
  "round_2_verification": {
    "stage_approved": true,
    "approved_at": "ISO8601时间戳",
    "auditors_recheck": [
      {
        "auditor_id": "auditor_a",
        "final_scores": {"total_score": 90},
        "verdict": "PASSED",
        "unresolved_issue_refs": []
      }
    ]
  }
}
```

审查报告检查项：

- `coverage`：all_source_read / events_preserved / arcs_preserved / ending_preserved
- `continuity`：causality / timeline / identity / states / knowledge_and_reveals
- `comic`：drawable_panels / dialogue_and_speakers / reading_order / pacing / text_density

报告必须带非空 `evidence`、真实 `reviewed_chapter_ids` 与 `script_hash`。机械字段通过 `review` 登记，程序不代写结论。

## 交付数据契约

`project.json` 只保存分镜生产所需数据：`source`、`script`、`reviews`、`script_lock`、`continuity_handover`。

不写 `art`、`layout`、`exports`、`final_review`。不存在页面尝试账本、图像哈希、预览尺寸或导出记录。

脚本锁定后再修改 `script`，`script_lock` 立即失效，`reviews` 一并作废，必须重跑三轮审查与阶段审查。
