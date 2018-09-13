---
layout:     post
title:      "tumblr的Java爬虫--单用户版本(二)"
subtitle:   " \"有关图片与视频链接的处理\""
date:       2018-08-11
author:     "Aziz"
header-img: "img/in-post/tumblr_crawl_java/tumblr_crawl_java.png"
tags:
    - java
    - crawl
---

> “Java的爬虫工具之图片与视频链接获取”


## 前言

图片与视频链接是通过jsoup来解析获取的，有关jsoup的介绍，在上期博文中已经有了，下面就看看具体如何解析tumblr的博文

## 解析方式

### 确认本地环境可以访问tumblr

1. 你得确认本地已经启用shadowsocks，并且可以访问到www.tumblr.com（默认已经会使用shadowsocks，不会的移步教程[自己在墙上搭梯子的方式](https://seolin.github.io/2018/08/16/world_out_of_wall/)）
2. 查看设置shadowsocks的本地http代理，本教程默认地址为127.0.0.1，端口为1087
3. 使用以下代码测试本地java进程是否能通过代理的方式访问tumblr
```
 public static void main(String[] args) {
        String urlStr = "www.tumblr.com";
        Document document = getDocByUrl(urlStr);
        System.out.println(document.toString());
    }

    /**
     * 获取链接，并且处理异常
     *
     * @param url 待获取的链接
     * @return
     */
    public static Document getDocByUrl(String url) {
        Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("127.0.0.1", 1087));
        Connection connection = Jsoup.connect(url).proxy(proxy);
        try {
            return connection.get();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
```
### 寻找一个用户，测试抓取图片链接
通过热门内容随意寻找一个图片，得到用户首页地址，例如：http://jonginssoo.tumblr.com/

分析网页源代码，可以看到包含图片的标签是 img ，如下所示
```
<div class="photo-data">
<div class="pxu-photo">
<img src="https://78.media.tumblr.com/779683d855af844c0e4a9efa66a815d5/tumblr_pextvaDX1J1rrhp2bo1_500.gif" width="500" height="231" data-highres="https://78.media.tumblr.com/779683d855af844c0e4a9efa66a815d5/tumblr_pextvaDX1J1rrhp2bo1_540.gif" data-width="540" data-height="250">
</div><!-- pxu-photo -->
<a class="tumblr-box" rel="post-178006751810" href="https://78.media.tumblr.com/779683d855af844c0e4a9efa66a815d5/tumblr_pextvaDX1J1rrhp2bo1_540.gif"></a>
</div><!-- photo-data -->
```
因此，可以通过img这个标签中的src属性来获取图片，使用jsoup来分析，代码如下
```
    public static void main(String[] args) {
        String urlStr = "http://jonginssoo.tumblr.com/";
        List<String> imageUrlStrList = new ArrayList<>();
        Document document = getDocByUrl(urlStr);
        imageUrlStrList.addAll(getImageUrlList(document));
        for (String imageUrlStr : imageUrlStrList) {
            System.out.println("imageUrlStr:" + imageUrlStr);
        }
    }
    
    /**
     * 获取图片链接列表
     * @param document 等地啊分析的文档
     * @return
     */
    public static List<String> getImageUrlList(Document document) {
        List<String> imageUrlStrList = new ArrayList<>();
        Elements elements = document.getElementsByTag("img");
        for (Element element : elements) {

            String imageUrlStr =  element.attr("src");
            if (imageUrlStr != null && imageUrlStr != "") {
                imageUrlStrList.add(imageUrlStr);
            }
        }
        return imageUrlStrList;
    }
    
```
再次仔细分析一下网页源代码可以发现，在data-highres属性中也有图片的链接，但是不能保证所有的图片中都有这个参数，因此可以首先通过获取data-highres中的url，如果不存在，则获取src中的url，getImageUrlList方法的代码如下
```
    /**
     * 获取图片链接列表
     * @param document 等地啊分析的文档
     * @return
     */
    public static List<String> getImageUrlList(Document document) {
        List<String> imageUrlStrList = new ArrayList<>();
        Elements elements = document.getElementsByTag("img");
        for (Element element : elements) {

            String imageUrlStr = element.attr("data-highres");
            if (StringUtils.isBlank(imageUrlStr)) {
                imageUrlStr = element.attr("src");
            }
            if (imageUrlStr != null && imageUrlStr != "") {
                imageUrlStrList.add(imageUrlStr);
            }
        }
        return imageUrlStrList;
    }
```
将取得的图片进行分析，可以发现data-highres中取得的图片相对来说质量比较高，并且其中不包含头像，小图标等图片，可以再将代码改为
```
    /**
     * 获取图片链接列表
     * @param document 等地啊分析的文档
     * @return
     */
    public static List<String> getImageUrlList(Document document) {
        List<String> imageUrlStrList = new ArrayList<>();
        Elements elements = document.getElementsByTag("img");
        for (Element element : elements) {

            String imageUrlStr = element.attr("data-highres");
            if (imageUrlStr != null && imageUrlStr != "") {
                imageUrlStrList.add(imageUrlStr);
            }
        }
        return imageUrlStrList;
    }
```
如果硬是要用src来下载图片的话，还需要注意上述的头像，小图标等辣鸡图片，已经要忍受图片有可能会很小的可能性（说到底，用data-highres取得可用图片的概率比较高）

### 寻找一个用户，测试抓取视频链接
> 需要注意的是，这里不下载外链分享的视频

通过热门内容随意寻找一个视频，得到用户首页地址，例如：http://yellowfeather84.tumblr.com/

分析网页源码，可以看到视频是在 iframe 的标签里面
```
  <div class="video-wrap"><div id='tumblr_video_container_177954389087' class='tumblr_video_container' style='width:500px;height:625px;'><iframe src='https://www.tumblr.com/video/earthstory/177954389087/500/' style='display:block;background-color:transparent;overflow:hidden' class='embed_iframe tumblr_video_iframe' scrolling='no' frameBorder='0' data-can-gutter data-can-resize data-width='500' data-height='625' width='500' height='625' allowfullscreen mozallowfullscreen webkitallowfullscreen></iframe></div></div>

```
但是访问链接" https://www.tumblr.com/video/earthstory/177954389087/500/ " 得到的不是可下载的视频，还是另外一个html页面。
因此，继续分析该页面，可以得到如下代码

```
<source src="https://earthstory.tumblr.com/video_file/t:Vl4fNU5daI9B1yPoQBECWw/177954389087/tumblr_pepvj7MZ0z1sq04bj/480" type="video/mp4">
```
所以还需要根据 source 的标签来得到真实的视频链接。代码如下所示
```
    public static void main(String[] args) {
        String urlStr = "http://yellowfeather84.tumblr.com/";
        Document doc = getDocByUrl(urlStr);
        List<String> crawlVideoUrlList = crawlNeedCrawlVideoUrl(doc);
    }

    public static List<String> crawlVideoUrl(Document doc) {
        List<String> videoCrawlUrlList = crawlNeedCrawlVideoUrl(doc);
        List<String> videoUrlList = new ArrayList<>();
        //真正的视频下载链接
        for (String videoCrawlUrl : videoCrawlUrlList) {
            Document videoDoc = getDocByUrl(videoCrawlUrl);
            if (videoDoc != null) {
                Elements videoElements = videoDoc.getElementsByTag("source");
                for (Element videoElement : videoElements) {

                    String videoUrl = videoElement.attr("src");
                    videoUrlList.add(videoUrl);
                    System.out.println("爬取的视频链接地址： " + videoUrl);
                }
            }
        }
        return videoUrlList;
    }

    /**
     * 抓去需要爬去的视频链接的链接
     *
     * @param doc
     * @return
     */
    public static List<String> crawlNeedCrawlVideoUrl(Document doc) {

        List<String> videoCrawlUrlList = new ArrayList<>();
        //爬图片和视频的链接
        Elements videoCrawlElements = doc.getElementsByTag("iframe");
        //视频预处理
        for (Element element : videoCrawlElements) {
            String videoCrawlUrl = element.attr("src");
            //通过正则过滤，判断是否为视频链接
            String pattern = "^.*video.*$";
            boolean isMatch = Pattern.matches(pattern, videoCrawlUrl);
            if (isMatch) {
                videoCrawlUrlList.add(videoCrawlUrl);
            }
        }
        return videoCrawlUrlList;
    }
```
## 结束语

本文介绍了如何爬去tumblr原始的视频与图片链接，为后面构建一个自动爬虫的系统做准备。