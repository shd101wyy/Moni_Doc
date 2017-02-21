# Moni 猫腻
![moni](/assets/moni.png)  
<center>[painting v0.0.1 by Yiyi Wang (not finished)] </center>  
<center> I don't know why I am writing this, but... I will continue</center>

<center> [中文文档](./README_CN.md)   </center>  

<center> [download this extension](https://chrome.google.com/webstore/detail/moni/hhmmkkgkbnnfdpmnckaicamnhfkolfej) </center>

**Moni** is a chrome extension that we are developing right now.  
I am planning to publish it to chrome web store in a month if I finish.  
This project is closed source (might be open sourced in the future).  
This repository will be used for issue posting and discussion.    

*The content below will be updated as the project development goes on.*

<!-- toc orderedList:0 depthFrom:2 depthTo:6 -->

* [Introduction](#introduction)
* [Script format](#script-format)
  * [Script function](#script-function)
  * [Script return](#script-return)
  * [Publish Script](#publish-script)
* [Example Scripts](#example-scripts)
  * [Twitter](#twitter)
  * [Twitch](#twitch)
* [UI](#ui)
  * [Moni page](#moni-page)
  * [Script page](#script-page)
* [RSS Support](#rss-support)

<!-- tocstop -->


## Introduction      
English **Moni**, 中文 **猫腻** comes from the word **monitor**.  
**Moni** is a page monitor extension.     

**Moni** is inspired by the chrome extension: [visualping](https://visualping.io/).  
However, **Moni** works differently from visualping in many ways.   

**Moni** will allow user to write script in javascript to help monitor webpage changes.  
In addition, user can publish their script to public (A remote server I am building), so other users can download the script and use it.  

The basic idea of **Moni** script is that each script will have a `target_url` field that indicates whether this script can be applied to this `target_url` webpage or not. If so, then the script can be applied and produce a **Moni** (monitor card) that shows information returned from the script.  


## Script format    
Sample scripts can be found at [Moni_Sample_Scripts](https://github.com/shd101wyy/Moni_Sample_Scripts).  
The script format is like below.  
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
      description: "description of this moni",
      // link: "(optional) where to jump to? eg: https://github.com",
      // favicon: "(optional) favicon? eg: https://github.com/favicon.ico",
      // media: {cover: "(optional) media to put"}
    })
  }
}
```

`target_url`: indicates for which url can this script be applied. You can write string or regular expression for this field.  
`name`: the name of this script.  
`description`: the description of this script.  
`keywords`: array of keywords.  
`link`: the developer page for this script.  
`public`: whether to publish to public or not.   
`script`: a function that should return a moni.  

### Script function  
The argument `event` that passed to `script` function has the following fields:   
`url`: current url that this script applied to.    
`cheerio`: [cheerio](https://github.com/cheeriojs/cheerio) library. you can use it to parse html.   
`$get`: ajax get function, `event.$get(opt, callback)`    

| Argument  |  Description |
|---|---|
| opt | the first argument   |
|  &nbsp;&nbsp;&nbsp;&nbsp;`url` | the url to get |
|  &nbsp;&nbsp;&nbsp;&nbsp;`dataType` | (optional) dataType to return, behave the same as jQuery ajax |  
| callback | callback function |
| &nbsp;&nbsp;&nbsp;&nbsp; `error` | is there error or not |
|  &nbsp;&nbsp;&nbsp;&nbsp; `body` | response body |  

`return`: used when you need to return something. As `script` is asynchronous, you need to call `event.return` to return a value.  

### Script return  
Call `event.return(false)` when you meet an `error` and don't want to update current moni.  
The `moni` object that you should return looks like this:  
```javascript
event.return({
  title: "Title you want to show",
  description: "description of this noty",
  link: "(optional) where to jump to? eg: https://github.com",
  favicon: "(optional) favicon? eg: https://github.com/favicon.ico",
  media: {cover: "(optional) media to put"}
})
```  
where `title`, `description` are required and can't be empty. The rest fields are optional.  

### Publish Script  
To publish your script, make sure your script has `public` set to `true`, and host your script somewhere like Github. Then search the url of your script on script page.   

![Screen Shot 2017-02-20 at 7.36.20 PM](http://i.imgur.com/K95hRyo.png)  

If you want to publish an updated version of script, make sure you have `version` incremented, then follow previous step.  

## Example Scripts  
I will introduce two example scripts:     

### Twitter
The script below tracks user's newest twitter.  
```javascript
{
  target_url: /^(https|http)\:\/\/twitter\.com\/([^\/]+)$/,
  name: "twitter update tracker",
  description: "track user twitter update",
  keywords: ["twitter"],
  link: 'https://github.com/shd101wyy/Moni_Sample_Scripts',
  version: "0.0.1",
  public: false,
  script: function (event) { // event = {body, url, cheerio}
    event.$get(event.url, (error, body)=> {
      if (error) return event.return(false)
      // eg: https://twitter.com/realDonaldTrump
      const $ = event.cheerio.load(body)
      const title = $('title').text().trim().replace(/\| Twitter$/, '')
      const description = $($('.js-tweet-text-container')[0]).text()
      const atName = title.match(/\(\@(.+)\)/)[1]
      const cover = `https://twitter.com/${atName}/profile_image?size=bigger`
      event.return({
        title,
        description,
        link: `https://twitter.com/${atName}`,
        favicon: `https://twitter.com/favicon.ico`,
        media: {cover}
      })
    })
  }
}
```  
The `target_url` field indicates for which url address can this script be used.  
The `public` field shows whether you want to share this script to others or not.  
For url `https://twitter.com/realdonaldtrump`, It will generate a **Moni** like this:  
![](/assets/example_moni_1.png)  

### Twitch
The script below tracks user's streaming status.  
```javascript
{
  target_url:/^(https|http)\:\/\/www\.twitch\.tv\/(.+)$/,
  name: "twitch live status check",
  description: "check twitch user/channel live status",
  keywords: ["twitch"],
  link: 'https://github.com/shd101wyy/Moni_Sample_Scripts',
  version: "0.0.1",
  public: false, // set to true if you want to publish this script so others can find it
  script: function(event) { // event = {body, url, cheerio, $get}
    event.$get(event.url, (error, body)=> {
      if (error) return event.return(false)
      const $ = event.cheerio.load(body)
      const title = $('meta[property="og:title"]').attr('content'),
            cover = $('meta[property="og:image"]').attr('content'),
            username = title.split(' ')[0]
      // http://streambadge.com/twitch/?username=sing_sing
      event.$get({
        url: `https://api.twitch.tv/kraken/streams/${username}?client_id=#{your_client_id}`,
        dataType: 'json',
      }, function(error, json) {
        if (error) {
          event.return(false) // error
        } else {
          event.return({
            title: title,
            description: json.stream ? 'live' : 'offline',
            link: event.url,
            media: {cover}
          })
        }
      })
    })
  }
}
```  
For url like `https://www.twitch.tv/sing_sing`, it will generate a **Moni** like this:  
![](/assets/example_moni_2.png)  

## UI  
The extension right now has two main pages.  
### Moni page  
**Moni page** shows all the **Monis** that you are using.    

![](/assets/moni_page.png)    
### Script page  
**Script page** shows all scripts that you wrote or downloaded from the public.    
You can search and install scripts on public that match current tab url.  
![](/assets/script_page.png)    

## RSS Support  
Moni supports very basic RSS subscription.  
To use RSS, you need to install `rss subscriber` script.  
![Screen Shot 2017-01-29 at 12.35.38 AM](</assets/Screen Shot 2017-01-29 at 12.35.38 AM.png>)
Then navigate to rss source, and click `apply` button.   
For example, for rss [anime-cloud tumblr](http://anime-cloud.tumblr.com/rss), it will generate a moni like this:    

![Screen Shot 2017-01-29 at 12.36.51 AM](</assets/Screen Shot 2017-01-29 at 12.36.51 AM.png>)   

Click its title will enter RSS page, which currently looks like this:  
![Screen Shot 2017-01-29 at 12.37.52 AM](</assets/Screen Shot 2017-01-29 at 12.37.52 AM.png>)
![Screen Shot 2017-01-29 at 12.38.01 AM](</assets/Screen Shot 2017-01-29 at 12.38.01 AM.png>)  

# MISC  
[download this extension](https://chrome.google.com/webstore/detail/moni/hhmmkkgkbnnfdpmnckaicamnhfkolfej)  
