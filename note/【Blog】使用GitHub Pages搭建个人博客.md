# 使用GitHub Pages搭建个人博客

> 对于程序员来说，个人博客不仅是记录、分享学习心得和感悟的地方，也是一张自己的技术名片。维护好个人博客页面对学习、工作和求职都能产生非常积极的影响。
>
> 【Blog】系列记录了我搭建个人博客的过程，我将介绍如何使用 Github Pages 搭建个人博客，并记录下搭建过程中遇到的问题及解决方法。

## GitHub Pages 是什么？

这里直接复制[官方文档](https://docs.github.com/zh/pages/getting-started-with-github-pages/what-is-github-pages)的介绍：

> GitHub Pages 是一项静态站点托管服务，它直接从 GitHub 上的存储库获取 HTML、CSS 和 JavaScript 文件，（可选）通过构建过程运行文件，然后发布网站。

简而言之，GitHub Pages 服务会根据你仓库中的内容渲染一个**静态网页**。就像传统的 B/S 架构应用一样，其他用户能通过特定的 URL 访问你的网页，而仓库中的内容充当了这个网页的后端部分。

## 如何开始？

创建一个仓库并命名为 `<username>.github.io`，并且开启 `README` 文件。

<p align="center">
  <img src="https://github.com/0x-0cd/0x-0cd.github.io/blob/main/large_file/images/note_0_0.png" alt="pic1" style="max-width: 40%; height: auto;">
</p>

然后通过 `https://<username>.github.io` 访问你的页面。

接下来像管理普通的代码仓库那样管理你的博客仓库，用 Markdown 撰写并发布博客内容。

## 一些建议

- 如果你的博客内容中可能包含大量图片、视频等二进制文件，建议使用[Git Large File Storage](https://docs.github.com/zh/repositories/working-with-files/managing-large-files/about-git-large-file-storage)来管理它们

- 如果你觉得页面不够好看，希望拥有更炫酷的博客主页，可以尝试使用[Hexo框架](https://hexo.io/zh-cn/)
