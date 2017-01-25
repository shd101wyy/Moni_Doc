# Moni 猫腻
![moni](/assets/moni.png)  
<center>[painting v0.0.1 by Yiyi Wang (not finished)] </center>  
<center> I don't know why I am writing this, but... I will continue</center>

**Moni** is a chrome extension that I am developing right now.  
I am planning to publish it to chrome web store in a month if I finish.  
This project is closed source (might be open sourced in the future).  
This repository will be used for issue posting and discussion.    

*The content below will be updated as the project development goes on.*

<!-- toc orderedList:0 depthFrom:2 depthTo:6 -->

* [Introduction](#introduction)
* [Script format](#script-format)
* [Example Scripts](#example-scripts)
  * [Twitter](#twitter)
  * [Twitch](#twitch)
* [UI](#ui)
  * [Moni page](#moni-page)
  * [Script page](#script-page)

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
TODO: write later.   
Not finalized yet.  

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
  version: "0.0.1",
  public: false,
  script: function (event) { // event = {body, url, cheerio}
      // eg: https://twitter.com/realDonaldTrump
      const $ = event.cheerio.load(event.body)
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
  version: "0.0.1",
  public: false, // set to true if you want to publish this script so others can find it
  script: function(event) { // event = {body, url, cheerio, $get}
    const $ = event.cheerio.load(event.body)
    const title = $('meta[property="og:title"]').attr('content'),
          cover = $('meta[property="og:image"]').attr('content'),
          username = title.split(' ')[0]
    // http://streambadge.com/twitch/?username=sing_sing
    event.$get({
      url: `https://api.twitch.tv/kraken/streams/${username}?client_id=##Change_to_your_client_id##`,
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

