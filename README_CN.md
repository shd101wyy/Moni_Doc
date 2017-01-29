# Moni 猫腻
![moni](/assets/moni.png)  
<center>[painting v0.0.1 by Yiyi Wang (not finished)] </center>  
<center> I don't know why I am writing this, but... I will continue</center>

<center> [English Doc](./README.md) </center>
<center> [下载该插件](https://chrome.google.com/webstore/detail/moni/hhmmkkgkbnnfdpmnckaicamnhfkolfej) </center>

**Moni** 是一款我们正在开发的 chrome 插件。  
这个项目目前是闭源的（将来可能会开源）。
这个 repository 将主要用于讨论问题和提出需求。

*以下内容会随着开发而进行改动*

<!-- toc orderedList:0 depthFrom:2 depthTo:6 -->

* [介绍](#介绍)
* [脚本格式](#脚本格式)
  * [脚本函数](#脚本函数)
  * [脚本返回](#脚本返回)
  * [发布你的脚本](#发布你的脚本)
* [例子](#例子)
  * [斗鱼直播](#斗鱼直播)
  * [Bilibili 更新提醒](#bilibili-更新提醒)
* [UI](#ui)
  * [Moni page](#moni-page)
  * [Script page](#script-page)
* [RSS 支持](#rss-支持)
* [杂项](#杂项)

<!-- tocstop -->


## 介绍      
英文 **Moni**, 中文 **猫腻** 来源于单词 **monitor**.  
**Moni** 是一款页面监控插件。      

**Moni** 由 chrome 插件 [visualping](https://visualping.io/) 启发而来。  
但是, **Moni** 和 visualping 的工作方式完全不同。

**Moni** 允许用户编写 javascript 脚本来帮助进行页面监控。  
并且，用户可以向其他用户分享脚本（基于一个我还在搭建的极其不稳定的服务器）。     

**Moni** 的基本想法就是每一个脚本会拥有一个 `target_url` 项决定了该脚本能否在当前的页面使用。如果可以使用，那么脚本将会被执行并且生成一个 **Moni**（监控卡片）用于显示由脚本返回的信息。

## 脚本格式    
脚本事例可以在 [Moni_Sample_Scripts](https://github.com/shd101wyy/Moni_Sample_Scripts) 找到。  
以下为脚本格式：    
```javascript
{
  target_url: "https://www.google.com",
  name: "my script",
  description: "description of your script",
  keywords: ["script"],
  // link: "(optional) link to your script introduction page",
  version: "0.0.1",
  public: false, // set to true if you want to publish this script so others can find it
  script: function(event) { // event = {url, cheerio, $get}
    event.return({
      title: "Title you want to show",
      description: "description of this noty",
      // link: "(optional) where to jump to? eg: https://github.com",
      // favicon: "(optional) favicon? eg: https://github.com/favicon.ico",
      // media: {cover: "(optional) media to put"}
    })
  }
}
```

`target_url`: 指定什么 url 可以被这个脚本所应用。  
`name`: 这个脚本的名字。   
`description`: 这个脚本的描述。    
`keywords`: 关键字数组。  
`link`: 脚本的介绍网站。    
`public`: 是否要将这个脚本公开。   
`script`: 脚本函数，需要返回一个 moni 对象。  

### 脚本函数   
传递至 `script` 的函数参数 `event` 拥有以下几个项：  
`url`: 当前脚本所应用的 url。     
`cheerio`: [cheerio](https://github.com/cheeriojs/cheerio) 裤. 你可以使用它来处理 html。  
`$get`: ajax get 函数, `event.$get(opt, callback)`    

| 参数  |  描述 |  
|---|---|
| opt | 第一个参数   |
|  &nbsp;&nbsp;&nbsp;&nbsp;`url` | get 的 url |
|  &nbsp;&nbsp;&nbsp;&nbsp;`dataType` | (optional) 返回的 dataType，和 jQuery ajax 一样 |  
| callback | 回调 function |
| &nbsp;&nbsp;&nbsp;&nbsp; `error` | 有没有 error |
|  &nbsp;&nbsp;&nbsp;&nbsp; `body` | response body |  

`return`: 用于函数的返回。因为 `script` 是异步的，你需要调用 `event.return` 来返回值。  

### 脚本返回    
调用 `event.return(false)` 当你遇到 `error` 的时候，并且你不想更新当前的 moni。  
`moni` 对象应该返回如下：
```javascript
event.return({
  title: "你想现实的标题",
  description: "这个 moni 的描述",
  link: "(optional) 跳转到哪里？ eg: https://github.com",
  favicon: "(optional) favicon？ eg: https://github.com/favicon.ico",
  media: {cover: "(optional) 图片路径"}
})
```  
`title`, `description` 是必须的，而且不能是空字符串。其余的可以省略。  

### 发布你的脚本    
在发布你的脚本之前，你需要创建／登陆你的 developer 账号。

![Screen Shot 2017-01-29 at 12.50.12 AM](</assets/Screen Shot 2017-01-29 at 12.50.12 AM.png>)  

点击 齿轮按钮 进入设置页，然后你就可以创建或者登陆你的账号了。  
登陆之后，你将会看到你的开发者 `token`。  
接着你就可以改写你的脚本，将 `public` 项设置为 `true` 来发布了。

## 例子
I will introduce two example scripts:     

### 斗鱼直播    
我将会介绍两个脚本的例子。
```javascript
{
  target_url: /^(https|http)\:\/\/www\.douyu\.com\/(.+)(\/*)$/,
  name: "斗鱼直播在线提醒",
  description: "提醒主播在线状态",
  keywords: ["douyu", "live"],
  link: "https://github.com/shd101wyy/Moni_Sample_Scripts",
  version: "0.0.1",
  public: false,
  script: function (event) { // event = {url, cheerio, $get}
    event.$get(event.url, (error, body)=> {  // 下载当前的页面
      if (error) return event.return(false)  // 发生错误，返回 false
      const $ = event.cheerio.load(body)     // 使用 cheerio 分析 html
      let live = true
      if (body.indexOf('上次直播') >= 0) {    // 说明不在直播
        live = false
      }
      const title = $('title').text().replace(' - 每个人的直播平台', '')
      const cover = $('#anchor-info .fl img').attr('src')
      event.return({                        // 最终给返回的 moni 对象
        title,
        description: (live? '直播中':'不在直播'),
        link: event.url,
        media: {cover}
      })
    })
  }
}
```  
例如对于 `https://www.douyu.com/dcboys`，该脚本将会生成以下的 **Moni**    

![Screen Shot 2017-01-29 at 1.19.50 AM](</assets/Screen Shot 2017-01-29 at 1.19.50 AM.png>)  

### Bilibili 更新提醒   
```javascript
{
  target_url: /^(http|https)\:\/\/((www\.bilibili\.com\/video\/av(\d+))|(space\.bilibili\.com\/(\d+)))|bangumi\.bilibili\.com\/anime\/(\d+)/,
  name: "Bilibili 订阅助手",
  description: "可用于订阅 bilibili 番剧，up主空间视频更新。",
  keywords: [
    "bilibili", "up", "up主"
  ],
  link: 'https://github.com/shd101wyy/Moni_Sample_Scripts',
  version: "0.0.1",
  public: false,
  script: function(event) { // event = {url, cheerio, $get}
    const http = event.url.match(/^(http|https)\:\/\//)[1]

    // 查看是否为 up 主的链接。例如 敖厂长的 space.bilibili.com/122879
    // space
    const spaceMatch = event.url.match(/^(http|https)\:\/\/space\.bilibili\.com\/(\d+)/)
    if (spaceMatch) {
      const id = spaceMatch[2]
      return event.$get({
        // 下面的 url 是 bilibili 的 api。返回数据为 json 形式。
        url: `http://space.bilibili.com/ajax/member/getSubmitVideos?mid=${id}&pagesize=1`,
        dataType: 'json'
      }, (error, json) => {
        const newestVideoData = json.data.vlist[0]
        const title = newestVideoData.title
        const description = newestVideoData.description
        event.return ({
          title: title,
          description: description,
          link: `http://www.bilibili.com/video/av${newestVideoData.aid}/`,
          media: {
            cover: newestVideoData.pic
          }
        })
      })
    }

    // 这个函数会根据 seasonId 来返回番剧的最新一集信息。  
    function getSeasonInfo(seasonId) {
      event.$get({
        url: `http://bangumi.bilibili.com/jsonp/seasoninfo/${seasonId}.ver`,
        dataType: 'text'
      }, (error, text) => {
        if (error)
          return event.return (false)

        text = text.replace(/^seasonListCallback\((.+)\)\;$/, (x, a) => a)
        const json = JSON.parse(text)
        const result = json.result
        const bangumiTitle = result.bangumi_title,
              seasonTitle = result.season_title
        const newestEpisode = result.episodes[0],
              av_id = newestEpisode.av_id,
              cover = newestEpisode.cover,
              url = newestEpisode.webplay_url,
              updateTime = (new Date(newestEpisode.update_time)),
              episodeTitle = '第' + newestEpisode.index + '话 ' + newestEpisode.index_title

        const title = bangumiTitle + ' ' + seasonTitle
        const description = `[${episodeTitle}](${url})`
        event.return ({
          title,
          description,
          link: `http://bangumi.bilibili.com/anime/${seasonId}`,
          favicon: `http://www.bilibili.com/favicon.ico`,
          media: {
            cover: cover
          }
        })
      })
    }

    // season
    // 例如链接 http://bangumi.bilibili.com/anime/3287/
    // 是动画 火影忍者
    const seasonMatch = event.url.match(/bangumi\.bilibili\.com\/anime\/(\d+)/)
    if (seasonMatch) {
      const seasonId = seasonMatch[1]
      return getSeasonInfo(seasonId) // 调用上面定义的函数
    }

    // av
    // 例如 http://www.bilibili.com/video/av8215024/  
    // 是动画 不良人
    const avId = event.url.match(/bilibili\.com\/video\/av(\d+)/)[1]
    const targetUrl = `${http}://app.bilibili.com/bangumi/avseason/${avId}.ver`

    event.$get({
      url: targetUrl,
      dataType: 'text'
    }, (error, text) => {
      if (error)
        return event.return (false)
      text = text.replace(/^seasonJsonCallback\((.+)\)\;$/, (x, a) => a)
      const json = JSON.parse(text)
      // 因为不是所有的 avId 都有 season 信息，所以如果失败，返回 false
      if (json && json.message !== 'success')
        return event.return (false)
      const result = json.result
      const seasonId = result.season_id
      return getSeasonInfo(seasonId) // 调用上面定义的函数
    })
  }
}
```   
对于链接 `http://space.bilibili.com/122879/#!/index`，该脚本将会生成以下 **Moni**：    

![Screen Shot 2017-01-29 at 1.26.14 AM](</assets/Screen Shot 2017-01-29 at 1.26.14 AM.png>)  

对于链接 `http://bangumi.bilibili.com/anime/5774/play#98565`，该脚本将会生成以下 **Moni**：

![Screen Shot 2017-01-29 at 1.28.04 AM](</assets/Screen Shot 2017-01-29 at 1.28.04 AM.png>)  

## UI  
现在该插件拥有以下页面  

### Moni page  
**Moni page** 将会显示你正在使用的 **Monis**。    

![](/assets/moni_page.png)    
### Script page  
**Script page** 显示你编写的脚本和从服务器下载的公共脚本。  
你可以搜索下载可以应用于当前页面的公共脚本。  

![](/assets/script_page.png)    

## RSS 支持    
Moni 支持非常简易的 RSS 订阅。    
要使用 RSS，你需要安装 `rss subscriber` 脚本。    
![Screen Shot 2017-01-29 at 12.35.38 AM](</assets/Screen Shot 2017-01-29 at 12.35.38 AM.png>)
打开一个 RSS 源的页面，例如上面的 `http://zhihu.com/rss`, 点击搜索，然后找到 `rss subscriber`，然后点击 `apply` 下载。   
例如对于知乎的 RSS，将会生成以下的 **Moni**：  

![Screen Shot 2017-01-29 at 1.33.00 AM](https://ooo.0o0.ooo/2017/01/29/588d9ae4a6616.png)  

点击标题进入 RSS 页：  

![Screen Shot 2017-01-29 at 1.34.25 AM](</assets/Screen Shot 2017-01-29 at 1.34.25 AM.png>)      
![Screen Shot 2017-01-29 at 1.34.30 AM](</assets/Screen Shot 2017-01-29 at 1.34.30 AM.png>)     

## 杂项  
[下载该插件](https://chrome.google.com/webstore/detail/moni/hhmmkkgkbnnfdpmnckaicamnhfkolfej)
