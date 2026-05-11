# 内置函数与 API 参考

## HTML 解析三件套

| 函数 | 缩写 | 返回值 | 说明 |
|---|---|---|---|
| parseDom(html, rule, baseUrl) | pd | String（智能补全） | 自动补全域名/http，返回第一个匹配 |
| parseDomForHtml(html, rule) | pdfh | String（原始） | 不处理域名，原始返回 |
| parseDomForArray(html, rule) | pdfa | Array\<String\> | 返回匹配列表，用于循环 |

```javascript
var items = pdfa(html, 'body&&.list&&li');
for (var i = 0; i < items.length; i++) {
  var title = pdfh(items[i], 'a&&Text');
  var href  = pd(items[i], 'a&&href', host);
  var pic   = pd(items[i], 'img&&data-original');
}
```

## 选择器语法

| 语法 | 说明 | 示例 |
|---|---|---|
| `&&` | 取子元素 | `'body&&.list&&li'` |
| `--` | 排除元素 | `'body--script&&a&&href'` |
| `tag,N` | 取第 N 个（0起） | `'a,2'` 取第3个 |
| `tag,-1` | 倒数取 | `'li,-1'` 最后一个 |
| `Text` | 文本内容 | `'h2&&Text'` |
| `Html` | 含标签 HTML | `'div&&Html'` |
| `attr` | 属性值 | `'img&&src'` |
| `[class*="val"]` | 模糊类名 | 比固定类名稳健 |
| `\|\|` | 或语法（先找前者） | `'#app\|\|#app2&&Text'` |
| `.js:expr` | 对结果 JS 处理 | `'href.js:input.replace(/\\//,"")'` |
| `js:代码` | 完全 JS 规则 | `'js:parseDom(html,"...") + "/page/1"'` |
| `tag,N:` | 从第N个取到末尾 | `'a,1:'` 跳过第一个 |
| `tag,N:M` | 取 N 到 M-1 | `'a,1:3'` 取第2、3个 |
| `:has(selector)` | 含指定子元素的父节点 | `'li:has(img)'` |

**旧版聚阅 Bug：** `pdfa` 深度嵌套有时只返回 1 条。绕过：用 `html.split('</article>')` 分割后逐段正则提取。

## 网络请求函数

| 函数 | 说明 |
|---|---|
| `fetch(url, opts)` | 标准请求。opts: {headers, body, method, timeout, withHeaders, withStatusCode, redirect} |
| `fetchPC(url, opts)` | PC UA 请求 |
| `request(url, opts)` | fetch 别名 |
| `post(url, {body: {k:v}})` | POST 表单 |
| `batchFetch(arr)` | 并发请求，最多 16 个/批，超出自动分批串行 |
| `fetchCookie(url, opts)` | 只获取 Set-Cookie |
| `fetchCodeByWebView(url, opts)` | WebView 加载获取 JS 渲染后源码 |
| `requireCache(url, hours)` | 远程 JS 缓存 eval，小时数内不重复请求 |

```javascript
// 带 header
var html = fetch(url, { headers: { 'User-Agent': parse.UA, 'Referer': parse.host } });
// 返回 header
var res = JSON.parse(fetch(url, { withHeaders: true }));
var ck = res.headers['Set-Cookie'][0];
// GBK 站点
var html = fetch(url, { headers: { 'content-type': 'text/html; charset=GBK' } });
// 禁止自动 Cookie
var html = fetch(url, { headers: { Cookie: '#noCookie#' } });
```

## col_type 显示样式

| col_type | 说明 |
|---|---|
| movie_3 / movie_3_marquee | 一行3列，圆角图片+标题（默认） |
| movie_2 | 一行2列 |
| movie_1 | 一行1列，描述较多时 |
| movie_1_left_pic | 图片在左 |
| movie_1_vertical_pic | 竖向图片，书籍/漫画封面 |
| text_1 ~ text_5 | 文本1~5列，支持 "" 和 '' 混排 |
| long_text / rich_text | 长文本 / 富文本 HTML |
| pic_1_full / pic_3 / pic_2 | 纯图片布局 |
| scroll_button | 横向滚动按钮（支持 HTML，多连续自动聚合） |
| flex_button | 流式自适应按钮 |
| blank_block | 空白块 |
| line | 分割线 |
| input | 单行输入框，extra:{type,defaultValue,onChange} |
| x5_webview_single | X5 浏览器组件，desc 控制高度/模式 |
| avatar | 头像样式（图片+标题+右侧desc） |
| card_pic_2 / card_pic_1 | 方形卡片 |
| icon_4 / icon_round_4 | 桌面图标样式，4列 |
| video | 视频组件 |

富文本前缀：`''''`（4单引号）标准富文本；`'''''`（5单引号）强制 HTML 渲染。

## URL 标签

追加在 URL 末尾改变行为，引擎自动剥除：

| 标签 | 说明 |
|---|---|
| `#noLoading#` | 不显示 loading 弹窗 |
| `#noHistory#` | 不记录足迹 |
| `#noRecordHistory#` | 不记录历史 |
| `#noAnim#` | 禁止跳转动画 |
| `#result#` | 获取服务器返回的纯 html 源码 |

## $ 工具函数

| 函数 | 说明 |
|---|---|
| `$.require(url)` | 导入模块。先执行路径文件代码，返回 $.exports 对象。若 .json 结尾则 JSON.parse |
| `$.importRequire(scope, url)` | 将模块引入到指定 scope 环境运行 |
| `$().rule(fn, args...)` | 生成新开页面 URL。在新页面中用 setResult(d) 设置内容 |
| `$().lazyRule(fn, args...)` | 生成懒加载 URL。用户点击后执行 fn 返回真实 URL |
| `$().image(fn, args...)` | 生成图片 URL。fn 返回 InputStream |
| `$().x5Lazy(fn, args...)` | X5 内核网页资源嗅探 URL |
| `$().webLazy(fn, args...)` | Webkit 内核网页资源嗅探 URL |
| `$().b64(str)` | Base64 编解码，解决 lazyRule 嵌套传值问题 |
| `$.type(val)` | 增强 typeof，识别 Array/null/Date 等 |
| `$.dateFormat(date, fmt)` | 日期格式化，支持 yyyy/MM/dd/HH/mm/ss |
| `$.extend(obj)` | 扩展 $ 工具，添加静态属性/方法 |
| `$().input(title, defaultVal, cb)` | 弹出单行输入框 |
| `$().confirm(title, okCb, cancelCb)` | 确认/取消框 |
| `$().select(title, options, columns, cb)` | 下拉/网格选择框 |
| `registerTask(name, url, intervalMs)` | 注册定时后台任务 |
| `unRegisterTask(name)` | 注销定时任务 |

## 其他常用内置

| 函数/变量 | 说明 |
|---|---|
| log(msg) | 调试日志 |
| MY_PAGE | 当前页码（需 type:'主页'+url:'fypage'） |
| MY_URL | 当前请求 URL（含替换后完整地址） |
| MY_HOME | MY_URL 计算的根域名 |
| MY_RULE.title / .find_rule | 当前规则信息 |
| MOBILE_UA / PC_UA | 内置 UA 变量 |
| sourcename | 当前规则名称字符串 |
| base64Encode(s) / base64Decode(s) | Base64 编解码 |
| encodeStr(s, charset) | 指定字符集 URL 编码（GBK 搜索用） |
| aesDecode(key, data) / aesEncode(key, data) | AES 加解密 |
| buildUrl(url, params) | GET 参数拼接 |
| refreshPage(false) | 刷新页面 |
| back(true/false) | 关闭当前页 |
| toast(msg) | 短提示 |
| confirm({title,content,confirm,cancel}) | 确认框 |
| setPageTitle(title) | 动态修改标题 |
| updateItem(id, obj) | 动态更新列表项 |
| addItemAfter(id, obj) | 在指定 id 项后插入 |
| deleteItem(id) | 删除指定 id 项 |
| deleteItemByCls(cls) | 按 cls 删除 |
| setPreResult(d) + setResult(d) | 先返回顶部元素再追加列表（海阔用） |
| findItemsByCls(cls) | 查找具有指定 cls 的所有项 |
