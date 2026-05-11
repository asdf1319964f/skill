# 聚阅/海阔视界 规则开发 Skill

> 让 Claude 精准理解海阔视界底层 API，帮你快速开发视频/漫画/小说解析规则。

## 这是什么

这是一个用于 **Claude Code / 聚阅 App** 的代理技能（Agent Skill），包含海阔视界规则开发所需的全套知识上下文：

- parse 对象骨架与静态分类模板
- 内置函数（pd / pdfh / pdfa / fetch）完整参考
- video:// / pics:// 协议详解
- 三层存储策略（juItem / storage0 / putMyVar）
- 高级模式：MACCMS 解密、CF 过盾、KVS Hash、VidHide 混淆

## 使用方式

### Claude Code 导入

在 Claude Code「代理技能」→「从 GitHub 导入」中填入：

```
https://github.com/asdf1319964f/skill
```

导入成功后，对话中提到**聚阅规则**、**海阔视界**、**hiker 规则**等关键词时会自动触发。

### 直接上传

将 `SKILL.md` 及 `references/` 目录打包成 zip，在对话中上传即可使用。

## 文件结构

```
SKILL.md                        # 主入口，架构速览 + 开发铁律
references/
├── 01-parse-skeleton.md        # parse 骨架 + 五维占位符 + URL 模板
├── 02-api-reference.md         # 全部内置函数、选择器语法、col_type、$ 工具
├── 03-protocols-storage.md     # 协议详解 + 三层存储 + 展开折叠 + token 初始化
└── 04-advanced-patterns.md     # 加密穿透 / CF 过盾 / 完整规则模板 + 铁律全表
agents/
└── openai.yaml                 # Agent 描述文件
```

## 触发关键词

- 聚阅规则开发
- 海阔视界规则 / hiker 规则
- 视频解析规则
- 聚阅源编写

## 开发铁律速查

| # | 规则 |
|---|---|
| 1 | 入口固定：type:'主页' + url:'fypage'，禁用 type:0 |
| 2 | lazyRule 内禁止 this，所有值显式传参 |
| 3 | 媒体资源必须挂 Referer 防盗链 |
| 4 | ;{...} 内 && 前后无空格 |
| 5 | Token/CK 一律用 juItem.set 持久化 |
| 6 | lazyRule 内禁用 while，用加载更多翻页 |
| 7 | 解析函数只用 return，禁用 setResult |

## 示例提示词

```
帮我开发一个聚阅规则，目标网站是 https://example.com，
需要支持视频列表、二级详情页和搜索功能。
```

---

适配引擎：海阔视界 2026+ (Rhino ES6 安全子集) / 聚阅 (ES5)
