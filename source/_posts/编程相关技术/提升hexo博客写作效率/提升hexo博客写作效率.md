---
title: 提升hexo博客写作效率
date: 2020-10-18 16:57:36
tags:
	- hexo
summary: 编写Hexo博客文章的步骤总结，配合Matery主题的特殊设置内容
---

1. 新建文章

```bash
hexo new post "文章标题"
```
为保证目录结构清晰易于查找，建议将生成的markdown文件放置在文件夹下
**<u>若有插入图片的需求需要在markdown文件的同级目录下新建同名文件夹，用于存储图片，文章中使用相对路径进行图片的引用（参考[甲醛传感器](https://github.com/1170300514/xyzhangblog/tree/master/source/_posts/IoT/TencentOSTiny%E7%94%B2%E9%86%9B%E4%BC%A0%E6%84%9F%E5%99%A8)文章的路径设置方式）</u>**

2. 在文章头信息中添加summary字段用于首页展示，相当于次级标题

   文章头信息有很多可配字段 详细如下：

   `Front-matter` 选项中的所有内容均为**非必填**的。但我仍然建议至少填写 `title` 和 `date` 的值。

   | 配置选项      | 默认值                         | 描述                                                         |
   | ------------- | ------------------------------ | ------------------------------------------------------------ |
   | title         | `Markdown` 的文件标题          | 文章标题，强烈建议填写此选项                                 |
   | date          | 文件创建时的日期时间           | 发布时间，强烈建议填写此选项，且最好保证全局唯一             |
   | author        | 根 `_config.yml` 中的 `author` | 文章作者                                                     |
   | img           | `featureImages` 中的某个值     | 文章特征图，推荐使用图床(腾讯云、七牛云、又拍云等)来做图片的路径.如: `http://xxx.com/xxx.jpg` |
   | top           | `true`                         | 推荐文章（文章是否置顶），如果 `top` 值为 `true`，则会作为首页推荐文章 |
   | cover         | `false`                        | `v1.0.2`版本新增，表示该文章是否需要加入到首页轮播封面中     |
   | coverImg      | 无                             | `v1.0.2`版本新增，表示该文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片 |
   | password      | 无                             | 文章阅读密码，如果要对文章设置阅读验证密码的话，就可以设置 `password` 的值，该值必须是用 `SHA256` 加密后的密码，防止被他人识破。前提是在主题的 `config.yml` 中激活了 `verifyPassword` 选项 |
   | toc           | `true`                         | 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能。前提是在主题的 `config.yml` 中激活了 `toc` 选项 |
   | mathjax       | `false`                        | 是否开启数学公式支持 ，本文章是否开启 `mathjax`，且需要在主题的 `_config.yml` 文件中也需要开启才行 |
   | summary       | 无                             | 文章摘要，自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要 |
   | categories    | 无                             | 文章分类，本主题的分类表示宏观上大的分类，只建议一篇文章一个分类 |
   | tags          | 无                             | 文章标签，一篇文章可以多个标签                               |
   | keywords      | 文章标题                       | 文章关键字，SEO 时需要                                       |
   | reprintPolicy | cc_by                          | 文章转载规则， 可以是 cc_by, cc_by_nd, cc_by_sa, cc_by_nc, cc_by_nc_nd, cc_by_nc_sa, cc0, noreprint 或 pay 中的一个 |

   以下为文章的 `Front-matter` 示例。

   ### 最简示例

   ```
   ---
   title: typora-vue-theme主题介绍
   date: 2018-09-07 09:25:00
   ---
   ```

   ### 最全示例

   ```
   ---
   title: typora-vue-theme主题介绍
   date: 2018-09-07 09:25:00
   author: 赵奇
   img: /source/images/xxx.jpg
   top: true
   cover: true
   coverImg: /images/1.jpg
   password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
   toc: false
   mathjax: false
   summary: 这是你自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要
   categories: Markdown
   tags:
     - Typora
     - Markdown
   ---
   ```

3. 文章内容撰写完成后使用hexo命令在本地运行

   ```bash
   hexo clean
   hexo generate
   hexo server
   ```

   查看无误后push到github触发CI生成网页

