---
layout: post
title:  "Google Analytics 和 Github Pages"
date:   2023-07-10 17:14:18 +0800
categories: web 
---
# Google Analytics 和 Github Pages
使用github pages 和 jekyll 可以快速搭建一个 blog. 那么问题来了，如何知道blog的访问数据呢？
Google Analytics 是非常专业的工具，如何使用最简单的方式集成到github pages 当中呢。

1. 获取 TrackID, 直接在 Google Analytics 当中注册账号，并设置跟踪网站既能获取对应的TrackID.
2. 获取跟踪代码，类似
```
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-XXXXXXXXXXXX');
</script>
```
3. 在github pages 当中新建 _includes 文件夹，新建html文件，将上述代码复制到其中
4. 在 _config.yml 当中添加配置：google_analytics: G-PBWHN2ZF80
5. push代码，在部署完成后，观察网页源代码。如果网页源码当中，出现了`Google tag` 代码说明，成功了。

这是一个非常简单的事情。如果稍微了解下前端知识，那就更简单了。