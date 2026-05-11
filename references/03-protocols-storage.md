# 视频/图片协议 + 三层存储 + 常见模式

## video:// 协议

视频直链协议格式：

```javascript
return 'video://' + realUrl + ';{User-Agent@' + ua + '&&Referer@' + host + '/}';
```

分号后为 Header 配置：`;{HeaderName@value&&HeaderName2@value2}`。
**注意：** `;{...}` 内 `&&` 前后无空格。

## pics:// 协议

图片/漫画协议格式：

```javascript
return 'pics://' + imgUrl1 + '&&' + imgUrl2 + '&&' + imgUrl3;

// 每张图可追加防盗链
var pics = [];
pics.push(url + '@headers={"Referer":"https://cdn.example.com/","User-Agent":"' + ua + '"}');
// 或简写
pics.push(url + '@Referer=https://cdn.example.com/');
return 'pics://' + pics.join('&&');
```

## #嗅探 兜底

当无法直接解析出视频直链时：

```javascript
return url + '#嗅探';
// 触发 App 内置嗅探器，从页面资源中自动匹配视频
```

## lazyRule 播放模式

**模式1：直接 video://（最稳定）**

```javascript
url: 'video://' + epHref + ';{User-Agent@' + parse.UA + '&&Referer@' + parse.host + '/}'
```

**模式2：lazyRule 内联（需请求解密）**

```javascript
url: $('#noLoading#').lazyRule(function(pUrl, ua, host) {
  var html = request(pUrl, { headers: { 'User-Agent': ua } });
  var slash = String.fromCharCode(92);
  var m = html.match(/https?:\/\/[^\s'"]+\.m3u8[^\s'".]*/);
  if (m) return 'video://' + m[0].split(slash).join('') + ';{User-Agent@' + ua + '&&Referer@' + host + '/}';
  return pUrl + '#嗅探';
}, detailUrl, parse.UA, parse.host)
```

lazyRule 内无法访问外部变量，所有值必须通过参数传入。

## 三层存储分层策略

```javascript
// 层1：juItem —— 跨 App 重启的凭证（token、动态域名、CK）
juItem.set('token', JSON.parse(fetch(authUrl)).data.token);
var token = juItem.get('token', '');

// 层2：storage0 —— 接口返回的大型分类/列表数据
if (!storage0.getMyVar('prefix' + 'fenlei')) {
  storage0.putMyVar('prefix' + 'fenlei', JSON.parse(fetch(apiUrl)).data);
}
var fenlei = storage0.getMyVar('prefix' + 'fenlei');

// 层3：putMyVar/getMyVar —— 用户当前操作状态
putMyVar('prefix' + 'tab', data.id);
var tab = getMyVar('prefix' + 'tab', '0');
```

防 key 冲突：所有 key 加固定规则前缀。key 禁止含 `://`。

## 分类按钮防重复刷新

点已选中项不刷页：

```javascript
url: $('#noLoading#').lazyRule(function(key, nowid, newid) {
  if (nowid != newid) {
    putMyVar(key, newid);
    refreshPage(false);
  }
  return 'hiker://empty';
}, 'prefix_tab', curId, data.id)
```

## 展开/折叠模板

```javascript
// 初始渲染：前N项，其余存 storage0
var allItems = buildItems();
storage0.putMyVar('prefix_hidden', allItems.slice(4));
allItems.slice(0, 4).forEach(function(item) { d.push(item); });

// 锚点
d.push({ col_type: 'blank_block', extra: { id: 'list_anchor' } });

// 展开/折叠按钮
d.push({
  title: '展开更多',
  extra: { id: 'fold_btn' },
  url: $('#noLoading#').lazyRule(function() {
    var fold = getMyVar('fold', '0');
    putMyVar('fold', fold == '0' ? '1' : '0');
    updateItem('fold_btn', { title: fold == '0' ? '折叠' : '展开更多' });
    if (fold == '0') {
      var hidden = storage0.getMyVar('prefix_hidden');
      addItemBefore('list_anchor', hidden);
    } else {
      deleteItemByCls('hidden_cls');
    }
    return 'hiker://empty';
  })
});
```

## token / deviceId 初始化范式

```javascript
_initToken: function() {
  var deviceId = juItem.get('deviceId', '');
  if (!deviceId) {
    deviceId = this._generateHex(32);
    juItem.set('deviceId', deviceId);
  }
  if (!juItem.get('token', '')) {
    var res = JSON.parse(fetch(this.host + '/api/user/traveler', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ deviceId: deviceId })
    }));
    juItem.set('token', res.data.token);
  }
},
_generateHex: function(len) {
  var chars = '0123456789abcdef', res = '';
  for (var i = 0; i < len; i++) res += chars[Math.floor(Math.random() * 16)];
  return res;
},
```

## 常用坑速查

| 问题 | 原因 | 解决 |
|---|---|---|
| addItemBefore 报错 | 锚点 id 不存在或被删除 | 先确认锚点已 push |
| deleteItemByCls 没效果 | cls 写在了 extra 外层 | 必须是 extra: { cls: 'xxx' } |
| updateItem 报找不到 | id 写在了 extra 外层 | 必须是 extra: { id: 'xxx' } |
| 展开后翻页重复插入 | 状态没持久化 | 翻页判断 MY_PAGE == 1 才初始渲染 |
| 二级函数 return 后 setResult 无效 | return 后代码不执行 | 删除 return 后的死代码 |
