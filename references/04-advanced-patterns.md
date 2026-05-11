# 高级模式：加密穿透 / CF 过盾 / KVS 解密 / 完整模板

## MACCMS V10 播放加密穿透（Law 39）

**特征：** 页面含 `player_aaaa?ids=...&key=...` 或 `MacPlayer`，`mac_url()` 加密。

### player_aaaa 双重解密

```javascript
var html = fetch(url, { headers: { 'User-Agent': ua } });
// 提取 MacPlayer 的 playurl
var macScript = pdfh(html, 'script&&Html');
var b64match = macScript.match(/MacPlayer\.PlayUrl\s*=\s*'([^']+)'/);
var decode1 = base64Decode(b64match[1]);
// 第一层：Base64 解码后是 eval 包装的 JS
var realUrl = decode1.match(/https?:\/\/[^\s'"]+\.m3u8[^\s'"]*/);
// 第二层：如果有进一步加密，可能需要 AES 解密
// 常见：Base64 → Hex → Xor → URL
```

### parseDom 加密

部分 MACCMS 站将播放地址放在 `parseDom` 函数参数里：

```javascript
// 页面里有 parseDom("加密字符串")
var m = html.match(/parseDom\(['"]([^'"]+)['"]\)/);
if (m) {
  var decoded = base64Decode(m[1]);
  var url = pd(decoded, 'video&&src', host);
}
```

## Cloudflare 过盾方案

### 方案1：x5_webview_single + fba 桥接

```javascript
d.push({
  col_type: 'x5_webview_single',
  desc: '100&&float',
  url: targetUrl,
  extra: {
    js: $.toString(function() {
      function wait() {
        if (document.querySelector('#challenge-success')) {
          var ck = fba.getCookie(location.href);
          fba.putVar('cf_cookie', ck);
          fba.toast('CF验证通过');
          fba.back();
        } else {
          setTimeout(wait, 500);
        }
      }
      wait();
    })
  }
});
```

然后在规则中读取：

```javascript
var ck = getVar('cf_cookie');
juItem.set('cf_cookie', ck);
// 后续请求带此 Cookie
var html = fetch(url, { headers: { Cookie: ck, 'User-Agent': ua } });
```

### 方案2：fetchCodeByWebView

```javascript
var html = fetchCodeByWebView(url, {
  headers: { 'User-Agent': ua },
  timeout: 15000
});
```

## 域名自愈系统（Law 14）

```javascript
// 主页函数顶部：先探测域名是否可用
var host = juItem.get('site_host', 'https://default.com');
// 测试
var test = fetch(host + '/', { timeout: 5000 });
if (!test || test.indexOf('404') > -1) {
  // 尝试替代域名列表
  var altHosts = ['https://mirror1.com', 'https://mirror2.com'];
  for (var i = 0; i < altHosts.length; i++) {
    var t = fetch(altHosts[i] + '/', { timeout: 5000 });
    if (t && t.indexOf('404') === -1) {
      host = altHosts[i];
      juItem.set('site_host', host);
      break;
    }
  }
}
```

## KVS 播放器 Hash 解密

KVS（Kernel Video Sharing）站：`get_file/{type}/{hash}/{path}`。Hash 通常是视频 ID + 分辨率 + 密钥的 MD5。

Hash 计算模式（逆向分析）：

```javascript
function kvsHash(videoId, resolution, seed) {
  var str = videoId + '_' + resolution + '_' + seed;
  // 实际算法需从 get_file URL 逆向
  // 观察规律：同视频不同清晰度 hash 不同
  return md5(str);
}
```

实际开发中，大多从播放页 `flashvars` 直接提取 `video_url` / `video_alt_url`，无需自己算 hash。

## VidHide Dean Edwards Unpacker 解密

VidHide 混淆视频地址（`eval(.....` 格式）：

```javascript
// 识别：页面含大量 eval(function(p,a,c,k,e,d)...)
// 提取 packed 内容
var packed = html.match(/eval\(function\(p,a,c,k,e,d\)\{[^}]+\}\([^)]+\)\)/);
if (packed) {
  // 方法1：js:expr 选择器自动解
  var url = pdfh(html, 'script&&Html.js:input.match(/https?:[^\s'"]+\.mp4[^\s'"]*/)');
  // 方法2：Dean Edwards Unpacker 手动解（对应 js 库）
}
```

## 完整规则骨架模板

```javascript
var parse = {
  作者: 'dev',
  版本: '20260510.V1',
  host: 'https://example.com',
  UA: 'Mozilla/5.0',
  页码: { '主页': 1, '分类': 1, '搜索': 1 },
  静态分类: {
    type: '主页',
    url: 'https://example.com/list/fyclass-fyarea--fyyear--fysort--fypage.html',
    class_name: '全部&电影&电视剧',
    class_url: '0&1&2',
    area_name: '全部&大陆&日韩',
    area_url: '&大陆&日韩',
    year_name: '全部&2026&2025',
    year_url: '&2026&2025',
    sort_name: '时间&热度',
    sort_url: 'time&hits'
  },
  主页: function() {
    var d = [], host = this.host, ua = this.UA;
    if (MY_PAGE == 1) {
      // 渲染分类行
      d.push({
        title: '分类', col_type: 'scroll_button',
        url: 'https://...',
      });
    }
    var url = host + '/list/' + getMyVar(host + 'c0', '0') + '/page/' + MY_PAGE + '/';
    var html = fetch(url, { headers: { 'User-Agent': ua } });
    var items = pdfa(html, 'body&&.list&&li');
    for (var i = 0; i < items.length; i++) {
      d.push({
        title:   pdfh(items[i], 'a&&title'),
        url:     pd(items[i], 'a&&href', host),
        pic_url: pd(items[i], 'img&&data-original'),
        desc:    pdfh(items[i], '.desc&&Text'),
        col_type: 'movie_3'
      });
    }
    return d;
  },
  二级: function(url) {
    var html = fetch(url, { headers: { 'User-Agent': this.UA } });
    var list = [];
    var eps = pdfa(html, '.episode&&a');
    for (var i = 0; i < eps.length; i++) {
      list.push({
        title: pdfh(eps[i], 'Text'),
        url:   'video://' + pd(eps[i], 'href', this.host) + ';{User-Agent@' + this.UA + '&&Referer@' + this.host + '/}'
      });
    }
    return {
      vod_name: pdfh(html, 'h1&&Text'),
      vod_pic:  pd(html, '.cover&&img&&src', this.host),
      desc:     pdfh(html, '.intro&&Text'),
      line:     ['默认线路'],
      list:     [list]
    };
  },
  解析: function(url) {
    var html = fetch(url, { headers: { 'User-Agent': this.UA } });
    var m = html.match(/https?:\/\/[^\s'"]+\.m3u8[^\s'".]*/);
    if (m) return 'video://' + m[0] + ';{User-Agent@' + this.UA + '&&Referer@' + this.host + '/}';
    return url + '#嗅探';
  },
  搜索: function(name, page) {
    var d = [], key = encodeURIComponent(name);
    var html = fetch(this.host + '/search/' + key + '/page/' + page + '/', {
      headers: { 'User-Agent': this.UA }
    });
    var items = pdfa(html, 'body&&.search-list&&li');
    for (var i = 0; i < items.length; i++) {
      d.push({/* same as homepage */});
    }
    return d;
  }
};
```

## 开发铁律全表（Law 速查）

| # | 规则 | 说明 |
|---|---|---|
| 1 | 入口固定化 | type:'主页'+url:'fypage'，禁用 type:0 |
| 2 | 作用域隔离 | lazyRule 内禁止 this，所有值显式传参 |
| 3 | 静默嗅探 | 直链用 video:// 协议，勿用 #嗅探 兜底 |
| 4 | 防盗链补全 | 媒体资源必须挂 Referer |
| 6 | 模糊选择器 | 类名随机时用 [class*=''] 替代固定类名 |
| 8 | Header 零空格 | ;{...} 内 && 前后无空格 |
| 11 | CF 破盾 | X5 沙箱 + fba.getCookie + juItem.set 持久化 |
| 14 | 域名自愈 | 主页顶部探测域名，juItem.set 持久化 |
| 16 | SID 强关联 | data-sid 匹配 paneID |
| 17 | 路径编码 | 非英文路径用 encodeURI |
| 18 | 持久化存储 | 动态数据用 juItem.set |
| 19 | 禁止循环分页 | lazyRule 内禁用 while 抓多页 |
| 21 | UI 字符增强 | Unicode 下标美化集数 |
| 22 | 指纹二次同步 | getVar → juItem.set + clearVar |
| 23 | 变量名避锋 | 不以 t/v/g 开头，用下划线 |
