---
name: {{SKILL_ID}}
description: {{DESCRIPTION}}
topic: {{TOPIC}}
createdAt: {{CREATED_AT}}
researchDepth: {{RESEARCH_DEPTH}}（分析了 {{NOTES_COUNT}} 篇内容）
---

# {{TOPIC}}

> 本 Skill 由 content-researcher 自动生成于 {{CREATED_AT}}
> 信息来源：{{PLATFORM}} 平台 {{NOTES_COUNT}} 篇内容的交叉验证
> 置信度：{{CONFIDENCE}}
> 上次更新：{{UPDATED_AT}}

## 领域概述

{{DOMAIN_OVERVIEW}}

<!-- 用 2~3 段话概括该主题的全貌，让 agent 快速建立认知框架 -->

## 核心知识体系

<!-- 按维度组织，每个维度是一个子章节 -->

### {{DIMENSION_1_TITLE}}

**置信度：** {{高/中/低}}（被 N 篇内容交叉验证）

{{DIMENSION_1_CONTENT}}

<!-- 重复以上结构，覆盖所有研究维度 -->

## 可执行工作流

<!-- 这是子 Skill 的核心价值：不仅告诉 agent "是什么"，还告诉它"怎么做" -->

### 工作流 1：{{WORKFLOW_NAME}}

**适用场景：** {{SCENARIO}}

**执行步骤：**

1. {{STEP_1}}
2. {{STEP_2}}
3. {{STEP_3}}
...

**预期产出：** {{EXPECTED_OUTPUT}}

<!-- 重复以上结构，覆盖所有可执行工作流 -->

## 工具与资源

<!-- 在研究过程中发现的有价值的工具、网站、账号等 -->

| 名称 | 类型 | 用途 | 推荐度 |
|------|------|------|--------|
| {{TOOL_NAME}} | {{网站/App/小程序}} | {{USE_CASE}} | {{高/中}} |

## 常见误区与避坑

<!-- 从内容中提炼的反面教训和常见错误 -->

1. **误区：** {{MISCONCEPTION}}
   **真相：** {{REALITY}}

## 时效性说明

<!-- 标注哪些信息可能随时间失效 -->

- {{TIME_SENSITIVE_ITEM_1}}：此信息基于 {{DATE}} 的平台规则，可能已更新
- {{TIME_SENSITIVE_ITEM_2}}：建议每 {{N}} 个月重新验证

## 信息来源

<!-- 列出本 Skill 的知识来源，支持溯源 -->

| # | 标题 | 作者 | 平台 | 时间 | 点赞 | 贡献维度 |
|---|------|------|------|------|------|---------|
| 1 | {{NOTE_TITLE}} | {{AUTHOR}} | {{PLATFORM}} | {{DATE}} | {{LIKES}} | {{DIMENSION}} |

## 知识缺口

<!-- 诚实标注研究中未能覆盖或信息不足的部分 -->

- {{GAP_1}}：当前信息不足以形成结论，建议后续深入研究
- {{GAP_2}}：仅有 1 篇来源，置信度低

---

*本 Skill 由 content-researcher 主 Skill 自动生成。如需更新，请要求 content-researcher 对主题 "{{TOPIC}}" 进行增量研究。*
