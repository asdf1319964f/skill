---
name: juyue-rule-dev
description: 聚阅/海阔视界规则开发。用于：开发/调试/维护聚阅或海阔视界 App 的 JavaScript 解析规则（视频/漫画/小说/听书源）。涵盖 parse 骨架、内置函数(fetch/pd/pdfh/pdfa)、$工具($.lazyRule/$().rule)、三层存储、video:///pics://协议、MACCMS/VidHide/KVS加密过盾、CF破盾、col_type显示样式、静态分类占位符、lazyRule沙箱、网页嗅探等。当用户问及海阔视界规则开发、聚阅源编写、视频解析规则、hiker规则等话题时触发。
---

# 聚阅/海阔视界 规则开发

基于海阔视界（HikerView）底层 Rhino JS 引擎的规则开发。聚阅是上层封装，API 通用。

## 架构速览

| 层 | 引擎 | 语法限制 |
|---|---|---|
| 海阔视界（新版 2026+） | 自研 Rhino | ES6 安全子集（const/let/箭头函数/模板字符串可用） |
| 聚阅（旧版引擎） | 继承相同引擎 | 全量 var，仅 ES5 |

**开发时统一以海阔视界 API 为基准**，聚阅特别注意额外规则。不确定目标平台时，全量用 var + ES5 写。

## 核心工作流

### 1. parse 对象骨架

所有规则必须封装在 parse 对象中。完整骨架见 references/parse-skeleton.md。快速模板：

```javascript
var parse = {
  作者: 'dev',
  版本: '20260510.V1',
  host: 'https://example.com',
  UA: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  页码: { '主页': 1, '分类': 1, '搜索': 1 },
  静态分类: { type: '主页', url: 'fypage', class_name: '...', class_url: '...' },
  主页:   function() { /* return d; */ },
  二级:   function(url) { /* return {vod_name, list...}; */ },
  解析:   function(url) { /* return "video://..."; */ },
  搜索:   function(name, page) { /* return d; */ },
};
```

**铁律：** 入口必须 type:'主页'+url:'fypage'。禁用 type:0。

### 2. 内置 API

完整参考见 references/api-reference.md。最常用三件套：

```javascript
pd(html, rule, baseUrl)     // 取第一个匹配，自动补全域名
pdfh(html, rule)            // 取第一个匹配，原始返回
pdfa(html, rule)            // 取所有匹配，返回 Array<String>
fetch(url, opts)            // HTTP GET/POST
```

选择器语法：`tag&&.class&&tag&&Text/Html/attr`，支持 `,N` 索引、`||` 或语法、`js:` 表达式。

### 3. 返回协议

| 内容类型 | 格式 |
|---|---|
| 视频直链 | `return 'video://' + url + ';{User-Agent@UA&&Referer@host/}'` |
| 漫画/图集 | `return 'pics://' + url1 + '&&' + url2` |
| 嗅探兜底 | `return url + '#嗅探'` |

### 4. 三层存储

juItem.set/get = 持久化（token/CK）；storage0.putMyVar/getMyVar = 会话（接口缓存）；putMyVar/getMyVar = 会话（状态）；putVar/getVar = 全局跨规则。key 加前缀防冲突，禁用 `://`。

### 5. $ 工具函数

$().lazyRule(fn, args...) — 懒加载解析；$().rule(fn, args...) — 新页面 setResult；$.require(url) — 模块导入。

## 参考文件索引

- **references/parse-skeleton.md** — parse 对象完整骨架 + 静态分类五维占位符 + URL 模板
- **references/api-reference.md** — 全部内置函数、选择器语法、col_type、URL标签、$工具、其他变量
- **references/protocols-storage.md** — video:///pics://协议详解 + 三层存储策略 + 分类按钮/展开折叠/动态默认值 + token初始化
- **references/advanced-patterns.md** — MACCMS V10穿透 / CF过盾 / 域名自愈 / KVS Hash / VidHide + 完整规则骨架模板 + 开发铁律全表

## 开发铁律（速查）

1. 入口固定化：type:'主页' + url:'fypage'
2. 作用域隔离：lazyRule 内显式传参，禁用 this
3. 防盗链：所有媒体资源挂 Referer
4. 选择器稳健：类名随机时用 [class*=''] 模糊匹配
5. Header 零空格：;{...} 内无多余空格
6. 持久化：Token/CK 一律 juItem.set
7. 禁止循环分页：lazyRule 内禁用 while，用加载更多
8. 变量名避锋：不以 t/v/g 开头，用下划线
9. 解析函数只用 return，禁用 setResult
