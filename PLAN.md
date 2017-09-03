# Moni 计划书


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [与以往社交平台的差别](#与以往社交平台的差别)
* [客户端 Client 介绍](#客户端-client-介绍)
	* [发布信息 Post](#发布信息-post)
* [服务器端 Server 介绍](#服务器端-server-介绍)

<!-- /code_chunk_output -->


Moni（猫腻）项目由 [GNUSocial](https://www.gnu.org/software/social/), [Mastodon](https://joinmastodon.org/), 以及原 [Moni](https://github.com/shd101wyy/Moni_Doc) 项目启发。   
目标在于打造一款 **去中心化**，**用户拥有数据**，**连接到世界**的社交平台。  

## 与以往社交平台的差别
传统的社交平台是中心化的，由**一个中心服务器面向多个用户**的。例如 Twitter 是一个公司面向不同的用户，用户需要在它的网站上注册并登录才可以使用。  
Moni 的主旨是**一个用户面向多个服务器**。
这里的一个用户指的就是你自己，多个服务器即是指其他第三方搭建的服务器。  
用户在他的 lifetime 内只拥有一个账号 ID。  
用户可以选择一个宿主（**host**）服务器来储存自己的发布的信息和用户设置。例如我选择了宿主服务器 `moni.io`，以及我的用户名 `shd101wyy`，那么我在世界中的名称就是 `@shd101wyy@moni.io`，其作用类似于电子邮件的地址。    
用户可以 export 自己发送过的信息，并且可以迁移（**migrate**）到其他的宿主服务器。迁移到其他服务器，例如 `bilibili.io`，那么我的世界名称将会变为 `@shd101wyy@bilibili.io`。     
用户同时可以浏览宿主服务器以外的服务器。    
整个 Moni 项目主要目的是提供 API 标准，第三方可以自行搭建客户端（**client**）和服务器（**server**）。Moni 项目将会自行包含一个 Client 和 Server 作为例子。  

## 客户端 Client 介绍  
客户端应支持一下功能：  

1. 显示世界上所有的 Server 列表。  
用户选择一个 Server 后，可以:
    1. 创建新的`世界`账号，并将所选服务器作为 host 服务器。
    1. 登陆到那个 server 作为 host server。

2. 发布信息 Post。  
这里的信息将以 `Markdown` 文档格式发布到 host server，并显示在你的 Timeline 上。
信息将会发布到:
    1. host server
    2. 关注过此用户（你）的其他 server

3. 转发（Repost） & 评论信息（Comment）
Repost 和 Comment 相似，唯一的区别是 Repost 将会将你发送的信息显示在你的 Timeline 上，和上述的 post 一样。Comment 则不会显示信息到你的 Timeline。

4. 赞（Like）
赞的信息将会显示到你的 Timeline 上。  

4. 订阅 (Subscribe)
你可以订阅一个 Post，那么当这个 Post 更新的时候或者收到评论的时候，你将会收到提醒。

6. 根据世界 id 关注其他用户。  

7. 显示你的关注者的 Post 

8. 运行 Moni 脚本。  
这个功能和 [Moni](https://github.com/shd101wyy/Moni_Doc) 插件类似，允许用户在本地运行脚本。并且私人订阅不被别人发现。  
脚本应当同样返回 `Markdown` 文档。                     
脚本的 Post 可以 `转发`，但无法 `评论`，`点赞`，`订阅`。  

9. 搜索信息，以及 `tag`。支持保存你的搜索存到 `my favorites`。  
例如，我对 `#宰执天下` 很感兴趣，那么我就搜索 `#宰执天下` 这个标签。我可以讲 `#宰执天下` 添加到我的 `favorites` 里面，这样我可以快速搜索。  
其作用和百度贴吧相似。  


### 发布信息 Post
用户发送的信息将通过 `markdown` 来呈现。  

为什么选择 `markdown`？:
1. 增加使用难度，淘汰低质量用户。
2. 便于显示。  

Client 应根据用户发送的 markdown 自动选择如何显示它。例如显示为 文章，还是显示为 图片。  

`markdown` 将会被扩展为支持 `tag` 以及 `at`。  
1. Tag: `#tag1 #tag2` 和 twitter 一样。  
2. At: `@shd101wyy` 和 twitter 一样。  

发布的 markdown 可以被设置为一下几类：
1. Public: Post to public timelines
2. Private: Do not post to public timelines 
3. Followers-only: Post to followers only
4. Direct: Post to mentioned users only

并且可以设置 media 内容为 sensitive 或者 insensitive。

其他用户可以 Repost 或者 Comment 一个 Post。服务器应显示一个 Post 的 Repost 以及 Comment 的多种排序方法，例如 General，Most popular，By time。显示方式类似于知乎。  


## 服务器端 Server 介绍  
对于 Moni 项目来说，一个 Server 的主要存在意义是作为 host server 来储存用户信息。  
一个 Server 应该提供一下功能：

1. 连接到其他的 server。这样世界类表中的 server 将会变多。  
2. 保存一个用户的 Posts 以及用户设置，并且支持 export。  
3. 当该 server 上的一个用户关注了其他 server 上的用户，该 server 应该同时收到那个用户的 post。  
4. 当其他 server 上的一个用户关注了你 server 上的用户，你的服务器应该将那个用户的 post 发送到那个 server。  
5. 提供热门 `tag`。
6. 显示本地 Timeline
7. 显示 Federated Timeline。（及包含其他 server 上的用户发送的信息。）  