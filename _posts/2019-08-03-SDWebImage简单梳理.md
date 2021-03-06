---
layout: post
title: "SDWebImage 梳理"
---
SDWebImage 梳理

### 1.cancel 取消正在下载的任务。
1. 当新图片被下载时，第一步的判断是是否有正在下载的任务，如果有的话取消当前下载任务并删除原先与view绑定的operation。

2. 如果是在正在从缓存中读取，取消读取的operation，若果是从是下载的任务，取消下载的operation。

3. 移除进行中的回调block (进度Blok 以及完成Bock)

4. 取消下载中任务(```SDWebImageDownloaderOperation```)；

5. 移除在下载管理类中记录的正在下载的operation(```SDWebImageDownloaderOperation```)。

6. 移除下载管理类中记录的任务（```SDWebImageCombinedOperation```);

7. 移除与View 关联的任务（```SDWebImageCombinedOperation```）

### 2.placeholder

   如果设置了占位图，给ImageView 设置占位图。

### 3.progress

   重置进度，设置进度回调block。combinedProgressBlock

### 4.和view关联正在下载的任务（```SDWebImageCombinedOperation```）

### 5.download

1. 验证url

2. ```[self.runningOperations addObject:operation];```下载管理类记录正在下载的任务 （```SDWebImageCombinedOperation```）（下载完成以后任务会被删除）；

3. 通过url去缓存查找（先查缓存 再查磁盘）

4. 缓存中没有找到需要去网络下载

* ```[self.URLOperations setObject:operation forKey:url];```下载管理类记录正在下载的任务（下载完成以后任务会被删除）

   *  在```SDWebImageDownloaderOperation```对象中记录回调Block（包括进度block 和完成blok）
   *  ```[self.downloadQueue addOperation:operation];```把下载任务加入队列，默认情况下最大并发是6

   5 .5、图片解码（分两种，下载过程中解码(先不提)、下载完成后解码）；
   *  当图片数据全部下载完后。会进行图片解码，图片解码是在一个异步串行队列中进行的。 



最后：这篇内容主要是我对阅读SDWebImage源码的一个简单的梳理总结，后续后写关于SDWebImage解码图片的内容。
