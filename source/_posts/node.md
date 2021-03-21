---
title: 使用nodejs开发一个命令行工具
date: 2021-03-21 09:29:57
tags: node 
descriptions: how to build Command line tool with nodejs
---

<!--more-->
使用的基本三方包
* [chalk](https://github.com/SBoudrias/Inquirer.js#readme)  文字颜色
* [inquirer](https://github.com/SBoudrias/Inquirer.js#readme) 命令行交互

开发一个自动搜索[漫画](https://www.leyuman.com/comic)并且下载的命令行工具

> 思路
 1. 分析搜索的API。获取api,method,还有返回的对象
 2. 返回的是html不是json。这里需要用到[cheerio](https://github.com/cheeriojs/cheerio/wiki/Chinese-README)来对html字符串进行解析来获得漫画的标题和地址。最后获取到一个数组Array
 3. 计算cpu的个数cpuNums，node创建多个子进程(cpuNums个进程)。数组Array切割成cpuNums个数组
 4. 子进程创建文件夹，图片下载
```
http.get(url).pipe(fs.createWriteStream(dest))
```
  5. 图片下载要控制并发，保证每时每刻只有n()个请求。不能用Promise.all。数组太多，程序可能就直接崩了。这里我限制每个进程每时每刻只并发20个图片请求。使用三方库[p-limit](https://github.com/sindresorhus/p-limit/blob/main/readme.md)
  6. 子进程完成下载任务后退出，并发送信息给主进程
  7. 主进程受到了所有子进程的信息后，自动退出

  按照以上思路，做出来一个小[demo](https://github.com/Stefanieissun/crawler-commic/tree/dev1)

  #### 不足
  1. 收集到的图片信息（标题和地址信息，目前是存放在内存数组中的，如果程序意外中断，再次启动就会出现重复下载的问题）
所以这里应该改成用redis来保存信息。用list来保存。每次pop走n个图片信息。子程序进行消费下载图片
2. 添加守护进程。实现显示下载速度和磁盘读写速度.cpu,内存等信息
3. 在联想轻薄本上（ubuntu系统）跑测试时。如果漫画图片很多(比如火影忍者)的时候，出现过子进程卡着不动的情况，在代码里没有看出明显bug。换了另一台笔记本(deepin)上测试。没有出现该问题。后来在Docker容器里跑也没有浮现
4. 显示进度条，而不是打印某某某下载完成。