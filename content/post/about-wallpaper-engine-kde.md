+++
title = '关于 wallpaper engine for kde'
date = 2025-05-28T23:06:46+08:00
draft = false
tags = ['linux']

+++

开端在大学时切换到 linux，想用上 win 的 wallpaper engine。  
遂开坑，当时写得还挺开心的，就是经常卡壳，需要找各种资料，毕竟里面涉及的东西还挺多的，当时也没有 AI。 

我记得卡了比较久的，骨骼蒙皮，各种噪音配合粒子生成，vulkan/opengl 的交互。  
当然最麻烦的还是解析一些二进制数据，要是没有 [ImHex](https://github.com/WerWolv/ImHex)，我估计直接放弃了。 

现在项目长期没有维护了，不过我最近打算开一些 3D 相关的新坑(大概是3D编辑相关)，就顺便把这个老项目翻新一下。  
不过重心不是重新实现 wallpaer engine 的功能，更多是偏向架构上的更改，比如跨进程的 dma-buf 交互。  

可能会顺手把 scene 内嵌视频的支持写了，当然是可能，这个还挺花时间的。  
当然进度可能会非常慢，想干的事太多了。

