# 平台适配配置

本文件定义 rednote-bootstrap 支持的内容平台及其特定的交互模式。

## 小红书 (xiaohongshu)

**状态：** 主力支持

### 平台特征

- URL: `https://www.xiaohongshu.com`
- 内容形态：图文笔记、视频笔记
- 关键特点：大量核心内容嵌入在图片中（非纯文本），需要截图 + 视觉理解
- 登录要求：需要登录才能完整浏览，支持 App 扫码 / 微信扫码
- 搜索入口：顶部搜索栏，支持关键词搜索

### 会话管理

```bash
# 首次登录（需要用户扫码）
agent-browser --headed --session-name xiaohongshu open "https://www.xiaohongshu.com"
# 等待用户扫码后保存状态
agent-browser state save {skill_dir}/xhs-auth-state.json

# 后续复用
agent-browser --headed --session-name xiaohongshu open "about:blank"
agent-browser state load {skill_dir}/xhs-auth-state.json
agent-browser open "https://www.xiaohongshu.com"
```

**注意：** `{skill_dir}` 需要替换为本技能的实际安装目录，即 SKILL.md 所在的目录。

### DOM 交互模式

**搜索流程：**
```bash
# 搜索框通常的 snapshot 标识
# textbox "搜索小红书" [ref=eXX]
agent-browser snapshot -i -s "header"
agent-browser click @搜索框ref
agent-browser fill @搜索框ref "搜索词"
agent-browser press Enter
agent-browser wait --load networkidle
```

**搜索结果解析：**
```bash
# 结果列表容器
agent-browser snapshot -i -s ".feeds-container"
# 每个结果结构：
#   link [ref=eXX]              ← 封面图（点击打开笔记）
#   link "标题文字" [ref=eXX]    ← 标题（点击打开笔记）
#   link "作者 日期" [ref=eXX]   ← 作者信息
#   generic "数字" [ref=eXX]     ← 点赞数
```

**笔记详情解析：**
```bash
# 详情弹窗容器
agent-browser snapshot -s ".note-detail-mask" -c
# 需要关注的文本节点：
#   - 标题（StaticText，紧跟在作者后面）
#   - 正文（StaticText，在标题下方的 generic 内）
#   - 标签（link "#xxx"）
#   - 评论（嵌套的 generic 结构）
```

**图片笔记翻页：**
```bash
# 图片导航点通常在 snapshot 中表现为一组 generic [ref=eXX] clickable
# 位置在图片区域下方，按顺序对应第 1、2、3...页
# 翻页：点击对应的导航点
agent-browser click @页码ref
agent-browser wait 1000
agent-browser screenshot ./page_N.png
```

### 注意事项

- 笔记内容主要在图片中：必须截图后用视觉理解来提取信息
- @ref 会在页面状态变化后失效：关闭笔记详情/滚动后需重新 snapshot
- 需要 scrollintoview 后再 click：不在视口内的元素点击无效
- 搜索结果会随滚动动态加载
- 点赞数可能是"9.8万"这种格式，需要解析为数字进行比较

### 内容质量信号

排序优先级（用于筛选高质量笔记）：
1. 点赞数 > 1000（说明内容被广泛认可）
2. 评论数 > 50（说明引发讨论）
3. 发布时间在近 6 个月内（时效性）
4. 作者有认证标识（专业性）
5. 标签与搜索主题高度相关（相关性）
