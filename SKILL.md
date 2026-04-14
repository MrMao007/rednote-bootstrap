---
name: rednote-bootstrap
description: >
  基于内容检索的小红书自动化账号运营Skill，通过广泛搜罗小红书的内容，构建出可复用的小红书运营Skill；应对时刻变化的账号运营需求，持续更新Skill流程、细节，形成个性化的账号运营技能包。预置了日常互动养号、赛道竞品分析、内容生成、运营数据分析等基础能力。当用户需要在小红书平台执行任意动作时，加载该技能。
---

# rednote-bootstrap

内容平台自进化研究 Skill —— 把"刷内容"变成"长知识"。

## 核心理念

```
用户提出主题 → 自动搜索内容平台 → 自适应深度采集 → 提炼为可执行子 Skill → 注册到导航表
                                                                         ↓
                                                            下次同类问题直接路由到子 Skill
```

## 文件结构

```
content-researcher/
├── SKILL.md                    # 本文件，主入口
├── registry.json               # 子 Skill 注册表（自动维护）
├── references/
│   └── platforms.md            # 平台适配配置
├── templates/
│   └── sub-skill-template.md   # 子 Skill 生成模板
└── generated-skills/           # 生成的子 Skill 存放目录
    ├── xiaohongshu-prohibited-words/
    │   └── SKILL.md
    └── .../
```

## 工作流程

### Phase 0：路由检查

在执行任何研究前，先检查 `registry.json` 是否已有匹配的子 Skill：

```bash
# 读取注册表
cat content-researcher/registry.json
```

匹配规则：
- 精确匹配：用户主题与已有子 Skill 的 `topic` 完全一致
- 模糊匹配：用户主题被已有子 Skill 的 `keywords` 覆盖超过 70%
- 如果匹配到，直接加载对应子 Skill 并执行，跳过后续研究流程
- 如果部分匹配，提示用户可基于已有子 Skill 增量更新

### Phase 1：主题分析与搜索词生成

收到用户主题后，进行多维度拆解：

**1. 主题分解**
将用户的主题拆解为 3~5 个子维度。例如：
- 用户主题："小红书违禁词检测机制"
- 子维度：违禁词分类清单、检测技术原理、隐形限流机制、自查方法工具、处罚规则与申诉

**2. 搜索词生成策略**
为每个子维度生成 2~3 个搜索词，策略如下：
- 直接词：子维度本身作为关键词（如"小红书违禁词清单"）
- 长尾词：加入场景或痛点（如"小红书笔记限流原因排查"）
- 反向词：从问题/失败角度搜索（如"小红书笔记没流量违规"）

生成搜索词后记录到工作日志：
```json
{
  "topic": "小红书违禁词检测机制",
  "dimensions": ["违禁词分类", "检测原理", "隐形限流", "自查工具", "处罚规则"],
  "search_queries": [
    {"dimension": "违禁词分类", "queries": ["小红书违禁词清单2026", "小红书敏感词分类汇总"]},
    ...
  ]
}
```

### Phase 2：内容采集（agent-browser）

**前置要求：** 必须先加载 agent-browser skill 文档。

```bash
agent-browser skills get agent-browser
```

**2.1 平台登录**

优先使用已保存的 session：
```bash
agent-browser --headed --session-name xiaohongshu open "https://www.xiaohongshu.com"
```

如无可用 session，需引导用户扫码登录后保存：
```bash
agent-browser state save ./xhs-auth-state.json
```

**2.2 搜索与采集循环**

对每个搜索词执行以下流程：

```
搜索词 → 搜索结果页 → 按点赞数筛选 Top 5~8 篇 → 逐一打开阅读 → 提取结构化信息
```

具体步骤：
```bash
# 1. 执行搜索
agent-browser fill @搜索框ref "搜索词"
agent-browser press Enter
agent-browser wait --load networkidle

# 2. 获取搜索结果列表
agent-browser snapshot -i -s ".feeds-container"

# 3. 按点赞数排序，选择 Top N 篇笔记
#    优先选择：点赞 > 1000、评论多、近期发布的笔记

# 4. 逐一打开笔记，提取内容
agent-browser click @笔记ref
agent-browser wait 2000
agent-browser snapshot -s ".note-detail-mask" -c
# 对于图片类笔记，需逐页截图并读取
agent-browser screenshot ./temp_note.png
```

**2.3 内容提取结构**

每篇笔记提取以下信息：
```json
{
  "title": "笔记标题",
  "author": "作者名",
  "publish_date": "发布时间",
  "likes": 9338,
  "content_type": "图文/视频",
  "key_points": ["要点1", "要点2", ...],
  "tools_mentioned": ["工具1", "工具2"],
  "actionable_tips": ["可执行建议1", ...],
  "dimension": "所属子维度",
  "novelty_score": 0.8,
  "source_url": "笔记链接"
}
```

### Phase 3：自适应深度控制

这是本 Skill 的核心创新点——不固定采集数量，而是根据信息饱和度动态决定何时停止。

**饱和度评估模型：**

每完成一轮搜索词的采集后，评估以下指标：

| 指标 | 计算方式 | 阈值 |
|------|---------|------|
| 信息新增率 | 本轮新增的独立要点数 / 本轮总提取要点数 | < 20% 则饱和 |
| 维度覆盖率 | 已有实质内容的维度数 / 总维度数 | > 80% 则充分 |
| 矛盾检测 | 不同笔记间是否存在互相矛盾的观点 | 有矛盾则需加深 |
| 权威性 | 是否包含官方账号或认证博主的内容 | 缺乏则需补充 |

**决策逻辑：**
```
IF 信息新增率 < 20% AND 维度覆盖率 > 80%:
    → 停止采集，进入 Phase 4
ELIF 信息新增率 < 20% AND 维度覆盖率 < 80%:
    → 针对未覆盖维度生成新搜索词，继续采集
ELIF 存在矛盾观点:
    → 针对矛盾点生成验证性搜索词，采集更多来源
ELSE:
    → 继续当前采集计划
```

**安全边界：**
- 单次研究最多执行 5 轮搜索（每轮 3~5 个搜索词）
- 单次研究最多阅读 30 篇笔记
- 单次研究耗时不超过 15 分钟
- 达到边界时强制进入 Phase 4，并在子 Skill 中标注"研究深度受限"

### Phase 4：知识蒸馏与子 Skill 生成

将采集到的全部结构化信息，按照 `templates/sub-skill-template.md` 的格式，蒸馏为一个可执行的子 Skill。

**蒸馏规则：**
1. 去重：合并不同笔记中重复的要点
2. 交叉验证：被 3 篇以上笔记提及的要点标记为"高置信度"
3. 冲突消解：对矛盾观点标注多方说法，不做强制裁决
4. 时效标注：标注信息的时效性（如"2026年规则"、"可能随平台更新变化"）
5. 可执行化：将笼统建议转化为具体的操作步骤

**生成子 Skill：**
```bash
# 子 Skill 保存路径
content-researcher/generated-skills/{topic-name-in-english}/SKILL.md
```

子 Skill 必须遵循 `templates/sub-skill-template.md` 的结构（见模板文件）。

### Phase 5：注册与导航更新

生成子 Skill 后，更新 `registry.json`：

```json
{
  "skills": [
    {
      "id": "xiaohongshu-prohibited-words",
      "topic": "小红书违禁词检测机制",
      "keywords": ["违禁词", "敏感词", "限流", "违规检测", "内容审核"],
      "path": "generated-skills/小红书违禁词检测/SKILL.md",
      "platform": "xiaohongshu",
      "created_at": "2026-04-13",
      "updated_at": "2026-04-13",
      "research_depth": "standard",
      "notes_analyzed": 12,
      "confidence": "high",
      "summary": "涵盖违禁词七大分类、检测机制原理、隐形限流、自查方法和推荐工具"
    }
  ]
}
```

并在终端向用户确认：
```
✅ 子 Skill 已生成：generated-skills/小红书违禁词检测/SKILL.md
📊 本次研究：分析了 12 篇笔记，覆盖 5 个维度，置信度：高
🔗 已注册到导航表，后续可直接调用
```

## 子 Skill 增量更新

当用户再次询问已有子 Skill 覆盖的主题时：

1. 加载已有子 Skill
2. 对比已有内容与用户新需求的差异
3. 如果用户需求在已有覆盖范围内 → 直接执行子 Skill
4. 如果有新的子维度 → 仅针对新维度执行 Phase 1~4，增量合并到子 Skill
5. 如果已有子 Skill 超过 30 天未更新 → 提示用户是否需要刷新（平台规则可能变化）

## 注意事项

- 所有浏览器操作必须通过 agent-browser 执行，先加载 skill 文档
- 小红书的内容主要在图片中，需要结合截图 + OCR 阅读
- 笔记 snapshot 中的 @ref 会在页面状态变化后失效，操作前需重新 snapshot
- 滚动到可见区域后再点击（scrollintoview + click）
- 部分笔记可能需要登录才能查看，确保 session 有效
- 生成的子 Skill 中不应包含版权争议内容，仅提炼方法论和结论性知识
