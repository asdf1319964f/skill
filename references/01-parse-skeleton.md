# parse 对象完整骨架 + 静态分类

## parse 对象标准结构

```javascript
var parse = {
  作者: 'dev',
  版本: '20260510.V1',
  host: 'https://example.com',
  UA: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  页码: { '主页': 1, '分类': 1, '搜索': 1 },
  静态分类: {
    type: '主页',
    url: 'fypage',
    class_name: '全部&电影&电视剧&动漫',
    class_url: '0&1&2&3',
  },
  主页: function() {
    var d = [];
    // 构建列表项，push 到 d
    // 每项格式: { title, url, pic_url, col_type: 'movie_3', desc }
    return d;
  },
  二级: function(url) {
    // url = 用户点击的列表项 URL
    var html = fetch(url, { headers: { 'User-Agent': this.UA } });
    var list = [];
    // 解析剧集列表
    return {
      vod_name: pdfh(html, 'h1&&Text'),     // 标题
      vod_pic:  pd(html, '.cover&&img&&src', this.host),  // 封面
      desc:     pdfh(html, '.intro&&Text'),   // 描述
      line:     ['默认线路'],                 // 线路数组
      list:     [list]                        // 二维数组，每线路一组
    };
  },
  解析: function(url) {
    // url = 用户点击的剧集 URL
    // 返回 video:// 协议串
    return 'video://' + realUrl + ';{User-Agent@' + this.UA + '&&Referer@' + this.host + '/}';
  },
  搜索: function(name, page) {
    // name = 搜索关键词，page 同 MY_PAGE
    var d = [];
    // GBK 站: var key = encodeStr(name, 'GBK');
    return d;
  }
};
```

## 静态分类 - 三种写法

| 写法 | type | url | 场景 |
|---|---|---|---|
| 纯矩阵自管理 | '主页' | 'fypage' | 主页内自渲染分类行，MY_PAGE 可用 |
| 带选单+翻页 | '主页' | 'hiker://empty##fyclass##fypage' | 顶部选单，MY_URL 带分类路径 |
| 无选单净模式 | 0 | 'fypage' | ⚠️ MY_PAGE 不注入，不推荐 |

## 五维占位符

```
url: "https://example.com/list/fyclass-fyarea-fysort--fyyear--fypage.html"
```

| 占位符 | 对应 name 字段 | 对应 url 字段 |
|---|---|---|
| fyclass | class_name | class_url |
| fyarea | area_name | area_url |
| fyyear | year_name | year_url |
| fysort | sort_name | sort_url |
| fypage | 引擎自动管理 | — |

**铁律：** name 与 url 数量严格一一对应（& 分隔）。"全部"选项 url 留空字符串。

## fypage 高级偏移

从 0 开始步长 20：`url: "list?start=fypage@-1@*20@&size=20"`
从 0 开始步长 1：`url: "list?page=fypage@-1@"`
首页特殊：`url: "list/fypage/[firstPage=https://example.com/list/]"`

fyAll：当一个维度选项太多时，class/area/year 的替换词都替换 fyAll。

## URL 模板示例

```
// MACCMS 路径型
"https://example.com/vod-show/fyclass-fyarea-fysort--fyyear--fypage---.html"

// 查询参数型
"https://api.example.com?type=fyclass&area=fyarea&year=fyyear&sort=fysort&page=fypage&_t=0"

// 纯路径拼接
"https://example.com/list/fyclass/fyarea/fyyear/fypage/"
```
