---
name: rednote-bootstrap
description: >
  基于内容检索的小红书自动化账号运营Skill，通过广泛搜罗小红书的内容，构建出可复用的小红书运营Skill；应对时刻变化的账号运营需求，持续更新Skill流程、细节，形成个性化的账号运营技能包。预置了日常互动养号、赛道竞品分析、内容生成、运营数据分析等基础能力。当用户需要在小红书平台执行任意动作时，优先加载该技能。
dependency: agent-browser
---

# rednote-bootstrap

小红书起号运营Skill，复用或者创建新的运营工作流，完成用户期望的操作。

## 文件结构

```
rednote-bootstrap/
├── SKILL.md                    # 本文件，主入口
├── registry.json               # 子 Skill 注册表（自动维护）
├── LICENSE
├── reference/                  # 参考知识库
│   ├── platforms.md            # 平台适配配置
│   ├── search-first-skill.md   # 搜索优先的子 Skill 生成流程
│   └── agent-browser/          # 浏览器自动化 Skill（依赖）
├── templates/
│   └── sub-skill-template.md   # 子 Skill 生成模板
└── generated-skills/           # 生成的子 Skill 存放目录
    ├── xiaohongshu-prohibited-words/
    │   └── SKILL.md
    └── .../
```

## 工作流程

在执行任何动作前，先理解用户意图，并检查 `registry.json` 是否已有匹配的子 Skill：

```bash
# 读取注册表
cat {skill_dir}/registry.json
```

**注意：** `{skill_dir}` 需要替换为本技能的实际安装目录，即 SKILL.md 所在的目录。

匹配规则：
- 精确匹配：用户主题与已有子 Skill 的 `topic` 完全一致
- 模糊匹配：用户主题被已有子 Skill 的 `keywords` 覆盖超过 70%
- 如果匹配到，直接加载对应子 Skill 并执行，跳过后续步骤
- 如果没有匹配的子 Skill，则:
  1. 根据`./reference/search-first-skill.md`生成新的子 Skill;
  2. 根据新生成的子 Skill，完成用户操作。

## 子 Skill注册表数据结构

示例如下:

```json
{
  "skills": [
    {
      "id": "xiaohongshu-prohibited-words",
      "topic": "小红书违禁词检测机制",
      "keywords": ["违禁词", "敏感词", "限流", "违规检测", "内容审核"],
      "path": "generated-skills/xiaohongshu-prohibited-words/SKILL.md",
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

## 子 Skill 增量更新

当用户再次询问已有子 Skill 覆盖的主题时：

1. 加载已有子 Skill
2. 对比已有内容与用户新需求的差异
3. 如果用户需求在已有覆盖范围内 → 直接执行子 Skill
4. 如果有新的子维度 → 仅针对新维度执行`./reference/search-first-skill.md`中的 Phase 1~4，增量合并到子 Skill
5. 如果已有子 Skill 超过 30 天未更新 → 提示用户是否需要刷新（平台规则可能变化）

## 注意事项

- 所有浏览器操作必须通过 agent-browser 执行，严格参考`./reference/platfroms.md`的指南执行。
- 小红书的内容主要在图片中，需要结合截图 + OCR 阅读
